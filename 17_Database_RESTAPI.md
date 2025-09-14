# 17. DATABASE & REST API - INTERVIEW GUIDE

## 🗄️ DATABASE FUNDAMENTALS & PERFORMANCE

### Database Selection Criteria
```javascript
// RDBMS (PostgreSQL, MySQL) - Khi nào dùng:
// ✅ ACID compliance, complex relationships
// ✅ Strong consistency, transactions
// ✅ Mature ecosystem, SQL expertise available

// NoSQL (MongoDB, Redis) - Khi nào dùng:
// ✅ Flexible schema, horizontal scaling
// ✅ Document/key-value structure fits data
// ✅ High throughput, eventual consistency OK

// Hybrid approach
const userProfile = await redis.get(`user:${userId}`); // Fast lookup
if (!userProfile) {
  const user = await pg.query('SELECT * FROM users WHERE id = $1', [userId]);
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user.rows[0]));
}
```

### Connection Pooling & Management
```javascript
// PostgreSQL với pg Pool
const { Pool } = require('pg');
const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  
  // Pool configuration
  max: 20,                    // Maximum connections
  min: 5,                     // Minimum connections
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 2000, // Wait 2s for connection
  maxUses: 7500,              // Close connection after 7500 queries
  
  // Retry configuration
  retryDelayMs: 1000,
  retryAttempts: 3,
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await pool.end();
  console.log('Pool has ended');
});

// MongoDB với Mongoose
const mongoose = require('mongoose');
mongoose.connect(process.env.MONGO_URL, {
  maxPoolSize: 10,      // Maximum connections
  minPoolSize: 5,       // Minimum connections
  maxIdleTimeMS: 30000, // Close after 30 seconds of inactivity
  serverSelectionTimeoutMS: 5000,
  heartbeatFrequencyMS: 10000,
  retryWrites: true,
  retryReads: true,
});
```

### Schema Design & Migrations
```javascript
// Migration với Sequelize
// migrations/001-create-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users', {
      id: {
        type: Sequelize.UUID,
        defaultValue: Sequelize.UUIDV4,
        primaryKey: true
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true,
        validate: { isEmail: true }
      },
      password_hash: {
        type: Sequelize.STRING,
        allowNull: false
      },
      created_at: {
        type: Sequelize.DATE,
        defaultValue: Sequelize.NOW
      },
      updated_at: {
        type: Sequelize.DATE,
        defaultValue: Sequelize.NOW
      }
    });
    
    // Add indexes
    await queryInterface.addIndex('users', ['email']);
    await queryInterface.addIndex('users', ['created_at']);
  },
  
  down: async (queryInterface) => {
    await queryInterface.dropTable('users');
  }
};

// TypeORM Migration
import { MigrationInterface, QueryRunner, Table, Index } from 'typeorm';

export class CreateUsersTable1635123456789 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          { name: 'id', type: 'uuid', isPrimary: true, generationStrategy: 'uuid' },
          { name: 'email', type: 'varchar', isUnique: true },
          { name: 'password_hash', type: 'varchar' },
          { name: 'created_at', type: 'timestamp', default: 'CURRENT_TIMESTAMP' },
          { name: 'version', type: 'int', default: 1 } // Optimistic locking
        ]
      })
    );
    
    await queryRunner.createIndex('users', new Index('idx_users_email', ['email']));
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

### Advanced Transactions & Concurrency
```javascript
// ACID Transactions với different isolation levels
const { Pool } = require('pg');
const pool = new Pool(config);

// Serializable transaction (highest isolation)
async function transferMoney(fromUserId, toUserId, amount) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN ISOLATION LEVEL SERIALIZABLE');
    
    // Check balance
    const fromBalance = await client.query(
      'SELECT balance FROM accounts WHERE user_id = $1 FOR UPDATE',
      [fromUserId]
    );
    
    if (fromBalance.rows[0].balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    // Update balances
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE user_id = $2',
      [amount, fromUserId]
    );
    
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE user_id = $2',
      [amount, toUserId]
    );
    
    // Record transaction
    await client.query(
      'INSERT INTO transactions (from_user, to_user, amount, created_at) VALUES ($1, $2, $3, NOW())',
      [fromUserId, toUserId, amount]
    );
    
    await client.query('COMMIT');
    return { success: true };
    
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// Optimistic Locking với Version Column
async function updateUserOptimistic(userId, updates, expectedVersion) {
  const result = await pool.query(
    `UPDATE users 
     SET name = $1, email = $2, version = version + 1, updated_at = NOW()
     WHERE id = $3 AND version = $4
     RETURNING *`,
    [updates.name, updates.email, userId, expectedVersion]
  );
  
  if (result.rows.length === 0) {
    throw new Error('Concurrent modification detected. Please refresh and try again.');
  }
  
  return result.rows[0];
}

// Deadlock handling
async function handleDeadlock(operation, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (error.code === '40P01' && i < maxRetries - 1) { // Deadlock detected
        await new Promise(resolve => setTimeout(resolve, Math.random() * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Query Optimization & Indexing
```javascript
// N+1 Query Problem
// ❌ Bad: N+1 queries
async function getBadUserPosts() {
  const users = await User.findAll(); // 1 query
  for (const user of users) {
    user.posts = await Post.findAll({ where: { userId: user.id } }); // N queries
  }
  return users;
}

// ✅ Good: Eager loading
async function getGoodUserPosts() {
  return await User.findAll({
    include: [{
      model: Post,
      attributes: ['id', 'title', 'created_at']
    }]
  });
}

// ✅ Good: DataLoader pattern
const DataLoader = require('dataloader');
const postLoader = new DataLoader(async (userIds) => {
  const posts = await Post.findAll({
    where: { userId: userIds },
    order: [['created_at', 'DESC']]
  });
  
  return userIds.map(userId => 
    posts.filter(post => post.userId === userId)
  );
});

// Database Indexing strategies
// Compound index for multiple conditions
CREATE INDEX idx_posts_user_status_date 
ON posts (user_id, status, created_at DESC);

// Partial index for specific conditions
CREATE INDEX idx_active_users 
ON users (created_at) WHERE status = 'active';

// Full-text search index
CREATE INDEX idx_posts_fulltext 
ON posts USING gin(to_tsvector('english', title || ' ' || content));

// Query execution plan analysis
EXPLAIN ANALYZE 
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at > '2023-01-01'
GROUP BY u.id, u.name
HAVING COUNT(p.id) > 5;
```

---

## 🔒 DATABASE SECURITY

### SQL Injection Prevention
```javascript
// ❌ Vulnerable to SQL injection
app.get('/users/:id', async (req, res) => {
  const { id } = req.params;
  const query = `SELECT * FROM users WHERE id = ${id}`; // Dangerous!
  const result = await pool.query(query);
  res.json(result.rows);
});

// ✅ Parameterized queries
app.get('/users/:id', async (req, res) => {
  const { id } = req.params;
  const result = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
  res.json(result.rows);
});

// ✅ Query builder (Knex.js)
const users = await knex('users')
  .select('id', 'name', 'email')
  .where('id', userId)
  .andWhere('status', 'active');

// NoSQL Injection prevention
// ❌ Vulnerable
const user = await User.findOne({ 
  email: req.body.email,
  password: req.body.password // Could be { $ne: null }
});

// ✅ Safe
const { email, password } = req.body;
if (typeof email !== 'string' || typeof password !== 'string') {
  return res.status(400).json({ error: 'Invalid input' });
}

const user = await User.findOne({ email, password: await bcrypt.hash(password, 10) });
```

### Row Level Security (PostgreSQL)
```sql
-- Enable RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own posts
CREATE POLICY user_posts_policy ON posts
    FOR ALL TO authenticated_role
    USING (user_id = current_setting('app.current_user_id')::int);

-- Policy: Public posts are visible to all
CREATE POLICY public_posts_policy ON posts
    FOR SELECT TO public
    USING (status = 'public');
```

### Secret Management
```javascript
// Environment-specific configuration
const config = {
  development: {
    database: {
      host: process.env.DB_HOST || 'localhost',
      port: process.env.DB_PORT || 5432,
      database: process.env.DB_NAME || 'myapp_dev',
      username: process.env.DB_USER || 'postgres',
      password: process.env.DB_PASS || 'password'
    }
  },
  production: {
    database: {
      connectionString: process.env.DATABASE_URL,
      ssl: {
        require: true,
        rejectUnauthorized: false
      }
    }
  }
};

// AWS Secrets Manager integration
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getDatabaseCredentials() {
  try {
    const secret = await secretsManager.getSecretValue({
      SecretId: process.env.DB_SECRET_ARN
    }).promise();
    
    return JSON.parse(secret.SecretString);
  } catch (error) {
    console.error('Error retrieving database credentials:', error);
    throw error;
  }
}
```

---

## ✅ DATA VALIDATION & SERIALIZATION

### Input Validation with Joi/Zod
```javascript
const Joi = require('joi');
const bcrypt = require('bcrypt');

// Joi validation schemas
const userSchema = Joi.object({
  name: Joi.string().min(2).max(50).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/).required(),
  age: Joi.number().integer().min(13).max(120),
  preferences: Joi.object({
    newsletter: Joi.boolean().default(false),
    theme: Joi.string().valid('light', 'dark').default('light')
  })
});

// Zod alternative
const { z } = require('zod');
const userSchemaZod = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  password: z.string().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  age: z.number().int().min(13).max(120).optional()
});

// Validation middleware
const validateUser = (req, res, next) => {
  const { error, value } = userSchema.validate(req.body);
  if (error) {
    return res.status(400).json({
      error: 'Validation failed',
      details: error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }))
    });
  }
  req.validatedData = value;
  next();
};

// Create user endpoint với validation
app.post('/api/users', validateUser, async (req, res) => {
  try {
    const { name, email, password, age, preferences } = req.validatedData;
    
    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);
    
    const user = await User.create({
      name,
      email,
      password_hash: passwordHash,
      age,
      preferences
    });
    
    // Serialize response (hide sensitive data)
    const responseUser = {
      id: user.id,
      name: user.name,
      email: user.email,
      age: user.age,
      preferences: user.preferences,
      created_at: user.created_at
      // password_hash excluded
    };
    
    res.status(201).json(responseUser);
  } catch (error) {
    if (error.code === '23505') { // PostgreSQL unique violation
      return res.status(409).json({
        error: 'Email already exists'
      });
    }
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

### Data Transformation & Serialization
```javascript
// Mongoose toJSON transformation
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  password_hash: String,
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  created_at: { type: Date, default: Date.now }
});

// Transform output
userSchema.methods.toJSON = function() {
  const user = this.toObject();
  
  // Remove sensitive fields
  delete user.password_hash;
  delete user.__v;
  
  // Transform _id to id
  user.id = user._id;
  delete user._id;
  
  return user;
};

// Sequelize serialization
const User = sequelize.define('User', {
  // ... fields
}, {
  defaultScope: {
    attributes: { exclude: ['password_hash'] }
  },
  scopes: {
    withPassword: {
      attributes: {} // Include all fields
    }
  }
});
```

---

## 🚀 ADVANCED REST API DESIGN

### Error Handling Standards
```javascript
// RFC 7807 Problem Details format
class APIError extends Error {
  constructor(type, title, status = 500, detail = null, instance = null) {
    super(title);
    this.type = type;
    this.title = title;
    this.status = status;
    this.detail = detail;
    this.instance = instance;
  }
}

// Error types
const ErrorTypes = {
  VALIDATION_ERROR: 'https://example.com/errors/validation-error',
  NOT_FOUND: 'https://example.com/errors/not-found',
  UNAUTHORIZED: 'https://example.com/errors/unauthorized',
  RATE_LIMIT: 'https://example.com/errors/rate-limit-exceeded'
};

// Global error handler
app.use((error, req, res, next) => {
  if (error instanceof APIError) {
    return res.status(error.status).json({
      type: error.type,
      title: error.title,
      status: error.status,
      detail: error.detail,
      instance: req.originalUrl,
      timestamp: new Date().toISOString(),
      trace_id: req.headers['x-trace-id']
    });
  }
  
  // Unhandled errors
  res.status(500).json({
    type: ErrorTypes.INTERNAL_ERROR,
    title: 'Internal Server Error',
    status: 500,
    instance: req.originalUrl,
    timestamp: new Date().toISOString()
  });
});

// Usage
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findByPk(req.params.id);
    if (!user) {
      throw new APIError(
        ErrorTypes.NOT_FOUND,
        'User not found',
        404,
        `User with ID ${req.params.id} does not exist`
      );
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

### Advanced Pagination
```javascript
// Cursor-based pagination (better for real-time data)
app.get('/api/posts', async (req, res) => {
  const { cursor, limit = 10, sort = 'created_at' } = req.query;
  
  let whereClause = {};
  if (cursor) {
    const decodedCursor = JSON.parse(Buffer.from(cursor, 'base64').toString());
    whereClause.created_at = { $lt: new Date(decodedCursor.created_at) };
  }
  
  const posts = await Post.find(whereClause)
    .sort({ [sort]: -1 })
    .limit(parseInt(limit) + 1); // Get one extra to check if there's more
  
  const hasMore = posts.length > limit;
  if (hasMore) posts.pop(); // Remove the extra post
  
  const nextCursor = hasMore && posts.length > 0
    ? Buffer.from(JSON.stringify({
        created_at: posts[posts.length - 1].created_at
      })).toString('base64')
    : null;
  
  res.json({
    data: posts,
    pagination: {
      has_more: hasMore,
      next_cursor: nextCursor,
      limit: parseInt(limit)
    },
    links: {
      self: req.originalUrl,
      next: nextCursor ? `${req.baseUrl}${req.path}?cursor=${nextCursor}&limit=${limit}` : null
    }
  });
});

// Composite cursor for complex sorting
function createCompositeCursor(item, fields) {
  const cursor = {};
  fields.forEach(field => {
    cursor[field] = item[field];
  });
  return Buffer.from(JSON.stringify(cursor)).toString('base64');
}
```

### Idempotency & Safe Retries
```javascript
const redis = require('redis');
const client = redis.createClient();

// Idempotency middleware
const idempotentRequest = (ttl = 3600) => {
  return async (req, res, next) => {
    const idempotencyKey = req.headers['idempotency-key'];
    
    if (!idempotencyKey && ['POST', 'PATCH', 'PUT'].includes(req.method)) {
      return res.status(400).json({
        error: 'Idempotency-Key header required for this operation'
      });
    }
    
    if (idempotencyKey) {
      const cacheKey = `idempotent:${idempotencyKey}`;
      const cached = await client.get(cacheKey);
      
      if (cached) {
        const response = JSON.parse(cached);
        return res.status(response.status).json(response.body);
      }
      
      // Store original response methods
      const originalSend = res.send;
      const originalJson = res.json;
      
      res.send = function(data) {
        client.setex(cacheKey, ttl, JSON.stringify({
          status: res.statusCode,
          body: data
        }));
        return originalSend.call(this, data);
      };
      
      res.json = function(data) {
        client.setex(cacheKey, ttl, JSON.stringify({
          status: res.statusCode,
          body: data
        }));
        return originalJson.call(this, data);
      };
    }
    
    next();
  };
};

// Usage
app.post('/api/payments', idempotentRequest(), async (req, res) => {
  const { amount, currency, card_token } = req.body;
  
  // Process payment - will only execute once per idempotency key
  const payment = await processPayment({ amount, currency, card_token });
  
  res.status(201).json(payment);
});
```

### Content Negotiation & Compression
```javascript
const compression = require('compression');

// Compression middleware with custom logic
app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  level: 6, // Compression level (1-9)
  threshold: 1024 // Only compress responses > 1KB
}));

// Content negotiation
app.get('/api/users/:id', async (req, res) => {
  const user = await User.findByPk(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  // Handle different content types
  res.format({
    'application/json': () => {
      res.json(user);
    },
    'application/xml': () => {
      const xml = `<?xml version="1.0"?>
        <user>
          <id>${user.id}</id>
          <name>${user.name}</name>
          <email>${user.email}</email>
        </user>`;
      res.set('Content-Type', 'application/xml');
      res.send(xml);
    },
    default: () => {
      res.status(406).json({
        error: 'Not Acceptable',
        supported: ['application/json', 'application/xml']
      });
    }
  });
});
```

---

## 📊 CACHING STRATEGIES

### Multi-Level Caching
```javascript
const Redis = require('ioredis');
const NodeCache = require('node-cache');

// L1: In-memory cache (fastest)
const memoryCache = new NodeCache({ stdTTL: 300 }); // 5 minutes

// L2: Redis cache (shared across instances)
const redis = new Redis(process.env.REDIS_URL);

// L3: Database (slowest but most reliable)

async function getCachedUser(userId) {
  // Try L1 cache first
  let user = memoryCache.get(`user:${userId}`);
  if (user) {
    console.log('Cache hit: Memory');
    return user;
  }
  
  // Try L2 cache
  const redisKey = `user:${userId}`;
  const cachedUser = await redis.get(redisKey);
  if (cachedUser) {
    user = JSON.parse(cachedUser);
    console.log('Cache hit: Redis');
    
    // Store in L1 for next time
    memoryCache.set(`user:${userId}`, user);
    return user;
  }
  
  // L3: Database query
  user = await User.findByPk(userId);
  if (user) {
    console.log('Cache miss: Database query');
    
    // Store in both caches
    memoryCache.set(`user:${userId}`, user);
    await redis.setex(redisKey, 3600, JSON.stringify(user)); // 1 hour
  }
  
  return user;
}

// Cache invalidation
async function updateUser(userId, updates) {
  const user = await User.update(updates, { where: { id: userId } });
  
  // Invalidate all cache levels
  memoryCache.del(`user:${userId}`);
  await redis.del(`user:${userId}`);
  
  return user;
}
```

### HTTP Caching Headers
```javascript
// ETag and conditional requests
const generateETag = (data) => {
  const hash = require('crypto').createHash('md5');
  hash.update(JSON.stringify(data));
  return hash.digest('hex');
};

app.get('/api/posts/:id', async (req, res) => {
  const post = await Post.findByPk(req.params.id, {
    include: ['author', 'comments']
  });
  
  if (!post) {
    return res.status(404).json({ error: 'Post not found' });
  }
  
  // Generate ETag based on post data
  const etag = generateETag(post);
  const clientETag = req.headers['if-none-match'];
  
  // Set cache headers
  res.set({
    'ETag': `"${etag}"`,
    'Cache-Control': 'public, max-age=300', // 5 minutes
    'Last-Modified': post.updated_at.toUTCString()
  });
  
  // Check if client has latest version
  if (clientETag === `"${etag}"`) {
    return res.status(304).end(); // Not Modified
  }
  
  res.json(post);
});

// Vary header for content negotiation
app.get('/api/users', (req, res) => {
  res.set('Vary', 'Accept, Accept-Encoding, Authorization');
  // ... response logic
});
```

---

## 🧪 TESTING STRATEGIES

### Unit Testing với Mocks
```javascript
// __tests__/services/userService.test.js
const UserService = require('../../src/services/userService');
const User = require('../../src/models/user');

// Mock the User model
jest.mock('../../src/models/user');

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('createUser', () => {
    it('should create a new user successfully', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'securePassword123'
      };
      
      const mockUser = { id: 1, ...userData, created_at: new Date() };
      User.create.mockResolvedValue(mockUser);
      
      const result = await UserService.createUser(userData);
      
      expect(User.create).toHaveBeenCalledWith({
        ...userData,
        password_hash: expect.any(String) // Hashed password
      });
      expect(result).toEqual(mockUser);
    });
    
    it('should throw error for duplicate email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'existing@example.com',
        password: 'password123'
      };
      
      const dbError = new Error('Email already exists');
      dbError.code = '23505'; // PostgreSQL unique violation
      User.create.mockRejectedValue(dbError);
      
      await expect(UserService.createUser(userData))
        .rejects.toThrow('Email already exists');
    });
  });
});

// Integration testing với test database
const request = require('supertest');
const app = require('../../src/app');
const { sequelize } = require('../../src/models');

describe('User API Integration Tests', () => {
  beforeAll(async () => {
    // Setup test database
    await sequelize.sync({ force: true });
  });
  
  afterAll(async () => {
    await sequelize.close();
  });
  
  beforeEach(async () => {
    // Clean database before each test
    await sequelize.truncate({ cascade: true });
  });
  
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'Test User',
        email: 'test@example.com',
        password: 'password123'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(Number),
        name: userData.name,
        email: userData.email
      });
      expect(response.body).not.toHaveProperty('password');
      expect(response.body).not.toHaveProperty('password_hash');
    });
  });
});
```

### Database Testing với Test Containers
```javascript
const { GenericContainer } = require('testcontainers');
const { Pool } = require('pg');

describe('Database Integration Tests', () => {
  let container;
  let pool;
  
  beforeAll(async () => {
    // Start PostgreSQL test container
    container = await new GenericContainer('postgres:13')
      .withEnvironment({
        POSTGRES_USER: 'test',
        POSTGRES_PASSWORD: 'test',
        POSTGRES_DB: 'testdb'
      })
      .withExposedPorts(5432)
      .start();
    
    const port = container.getMappedPort(5432);
    
    pool = new Pool({
      host: 'localhost',
      port,
      user: 'test',
      password: 'test',
      database: 'testdb'
    });
    
    // Run migrations
    await pool.query(`
      CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100),
        email VARCHAR(100) UNIQUE,
        created_at TIMESTAMP DEFAULT NOW()
      )
    `);
  });
  
  afterAll(async () => {
    await pool.end();
    await container.stop();
  });
  
  it('should perform database operations', async () => {
    // Insert test data
    await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2)',
      ['Test User', 'test@example.com']
    );
    
    // Query data
    const result = await pool.query('SELECT * FROM users WHERE email = $1', ['test@example.com']);
    
    expect(result.rows).toHaveLength(1);
    expect(result.rows[0]).toMatchObject({
      name: 'Test User',
      email: 'test@example.com'
    });
  });
});
```

---

## 📈 MONITORING & OBSERVABILITY

### Database Performance Monitoring
```javascript
const prometheus = require('prom-client');

// Database metrics
const dbConnectionsGauge = new prometheus.Gauge({
  name: 'db_connections_active',
  help: 'Number of active database connections',
  labelNames: ['pool_name']
});

const queryDurationHistogram = new prometheus.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query execution time',
  labelNames: ['query_type', 'table'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5, 10]
});

const queryErrorsCounter = new prometheus.Counter({
  name: 'db_query_errors_total',
  help: 'Number of database query errors',
  labelNames: ['error_type', 'table']
});

// Query instrumentation
function instrumentQuery(pool) {
  const originalQuery = pool.query.bind(pool);
  
  pool.query = async function(...args) {
    const startTime = Date.now();
    const [queryText] = args;
    
    // Extract table name from query
    const tableMatch = queryText.match(/(?:FROM|INTO|UPDATE|DELETE\s+FROM)\s+([a-zA-Z_][a-zA-Z0-9_]*)/i);
    const table = tableMatch ? tableMatch[1] : 'unknown';
    
    try {
      const result = await originalQuery(...args);
      
      // Record successful query metrics
      const duration = (Date.now() - startTime) / 1000;
      queryDurationHistogram
        .labels({ query_type: 'select', table })
        .observe(duration);
      
      return result;
    } catch (error) {
      // Record error metrics
      queryErrorsCounter
        .labels({ error_type: error.code || 'unknown', table })
        .inc();
      
      throw error;
    }
  };
}

// Connection pool monitoring
setInterval(() => {
  dbConnectionsGauge
    .labels({ pool_name: 'main' })
    .set(pool.totalCount - pool.idleCount);
}, 5000);

// Health check endpoint
app.get('/health', async (req, res) => {
  const checks = [];
  
  // Database connectivity
  try {
    await pool.query('SELECT 1');
    checks.push({ name: 'database', status: 'healthy' });
  } catch (error) {
    checks.push({ 
      name: 'database', 
      status: 'unhealthy', 
      error: error.message 
    });
  }
  
  // Redis connectivity
  try {
    await redis.ping();
    checks.push({ name: 'redis', status: 'healthy' });
  } catch (error) {
    checks.push({ 
      name: 'redis', 
      status: 'unhealthy', 
      error: error.message 
    });
  }
  
  const allHealthy = checks.every(check => check.status === 'healthy');
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks
  });
});
```

### Structured Logging
```javascript
const winston = require('winston');

// Create logger with structured format
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'user-api',
    version: process.env.APP_VERSION || '1.0.0'
  },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Add console transport for development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Database query logging middleware
function logQuery(query, params, duration, error = null) {
  const logData = {
    event: 'database_query',
    query: query.replace(/\s+/g, ' ').trim(),
    duration_ms: duration,
    param_count: params?.length || 0
  };
  
  if (error) {
    logger.error('Database query failed', {
      ...logData,
      error: error.message,
      error_code: error.code
    });
  } else if (duration > 1000) { // Slow query threshold
    logger.warn('Slow database query detected', logData);
  } else {
    logger.debug('Database query executed', logData);
  }
}

// Request logging middleware
app.use((req, res, next) => {
  const startTime = Date.now();
  const traceId = req.headers['x-trace-id'] || require('uuid').v4();
  
  req.traceId = traceId;
  res.set('X-Trace-ID', traceId);
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    
    logger.info('HTTP request completed', {
      event: 'http_request',
      trace_id: traceId,
      method: req.method,
      url: req.originalUrl,
      status_code: res.statusCode,
      duration_ms: duration,
      user_agent: req.headers['user-agent'],
      ip: req.ip
    });
  });
  
  next();
});
```

---

## 🎯 INTERVIEW QUESTIONS & SCENARIOS

### **1. "Giải thích N+1 query problem và cách khắc phục?"**

```javascript
// ❌ N+1 Problem
const users = await User.findAll(); // 1 query
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } }); // N queries
}

// ✅ Solutions:
// 1. Eager loading
const users = await User.findAll({ include: Post });

// 2. DataLoader (batching)
const postLoader = new DataLoader(userIds => 
  Post.findAll({ where: { userId: userIds } })
);

// 3. Single query with JOIN
const result = await sequelize.query(`
  SELECT u.*, p.* FROM users u 
  LEFT JOIN posts p ON u.id = p.user_id
`);
```

### **2. "Database transaction isolation levels khác nhau gì?"**

```
READ UNCOMMITTED:
- Dirty reads có thể xảy ra
- Performance cao nhất
- Ít khi sử dụng trong production

READ COMMITTED:
- Không có dirty reads
- Non-repeatable reads có thể xảy ra
- Default trong PostgreSQL

REPEATABLE READ:
- Không có dirty và non-repeatable reads
- Phantom reads có thể xảy ra
- Tốt cho reporting

SERIALIZABLE:
- Isolation cao nhất
- Không có anomalies
- Performance thấp nhất
```

### **3. "Cách handle database connection pooling?"**

```javascript
// Connection pool configuration
const pool = new Pool({
  max: 20,                    // Maximum connections
  min: 5,                     // Minimum idle connections
  idleTimeoutMillis: 30000,   // Close idle connections
  connectionTimeoutMillis: 2000, // Connection timeout
  acquireTimeoutMillis: 60000,   // Pool acquire timeout
  
  // Health check query
  testQuery: 'SELECT 1',
  
  // Retry configuration
  retryDelayMs: 1000,
  retryAttempts: 3
});

// Monitor pool health
setInterval(() => {
  console.log(`Pool stats: 
    Total: ${pool.totalCount}
    Idle: ${pool.idleCount}
    Waiting: ${pool.waitingCount}
  `);
}, 10000);
```

### **4. "REST API versioning strategies?"**

```javascript
// 1. URI Versioning (most common)
app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);

// 2. Header Versioning
app.use((req, res, next) => {
  const version = req.headers['api-version'] || 'v1';
  req.apiVersion = version;
  next();
});

// 3. Accept Header Versioning
app.use((req, res, next) => {
  const accept = req.headers.accept;
  if (accept?.includes('application/vnd.api+json;version=2')) {
    req.apiVersion = 'v2';
  }
  next();
});

// 4. Query Parameter (not recommended)
// GET /api/users?version=2
```

### **5. "Cách implement soft delete và tại sao dùng?"**

```javascript
// Sequelize soft delete
const User = sequelize.define('User', {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
  deleted_at: DataTypes.DATE
}, {
  paranoid: true, // Enable soft delete
  deletedAt: 'deleted_at'
});

// Usage
await user.destroy(); // Sets deleted_at timestamp
const users = await User.findAll(); // Excludes soft-deleted
const allUsers = await User.findAll({ paranoid: false }); // Includes deleted

// Manual implementation
const softDelete = {
  delete: async (id) => {
    return await User.update(
      { deleted_at: new Date() },
      { where: { id, deleted_at: null } }
    );
  },
  
  restore: async (id) => {
    return await User.update(
      { deleted_at: null },
      { where: { id } }
    );
  }
};

// Benefits:
// ✅ Data recovery capability
// ✅ Audit trails and compliance
// ✅ Referential integrity preservation
// ❌ Increased storage usage
// ❌ More complex queries
```

---

## ✅ COMPREHENSIVE CHECKLIST

### **Database Fundamentals:**
- [ ] RDBMS vs NoSQL selection criteria
- [ ] Connection pooling configuration
- [ ] Transaction isolation levels
- [ ] ACID properties implementation
- [ ] Indexing strategies và query optimization
- [ ] Migration và rollback procedures

### **Security:**
- [ ] SQL/NoSQL injection prevention
- [ ] Parameterized queries
- [ ] Row-level security
- [ ] Secret management
- [ ] TLS/SSL configuration

### **API Design:**
- [ ] RESTful principles
- [ ] HTTP status codes
- [ ] Error handling standards (RFC 7807)
- [ ] Pagination strategies (offset vs cursor)
- [ ] Idempotency implementation
- [ ] API versioning approaches

### **Performance:**
- [ ] Query optimization techniques
- [ ] N+1 query prevention
- [ ] Multi-level caching
- [ ] HTTP caching headers
- [ ] Connection pooling tuning

### **Testing:**
- [ ] Unit testing với mocks
- [ ] Integration testing
- [ ] Database testing strategies
- [ ] API contract testing

### **Production Concerns:**
- [ ] Monitoring và observability
- [ ] Structured logging
- [ ] Health checks
- [ ] Graceful shutdown
- [ ] Error tracking

---

## 🚀 PRACTICAL EXERCISES

1. **Build Complete CRUD API** với authentication, validation, pagination
2. **Implement Caching Strategy** - Redis + in-memory + HTTP caching
3. **Database Migration System** - Version control, rollback, CI/CD integration  
4. **Performance Optimization** - Index optimization, query analysis
5. **Security Hardening** - Input validation, rate limiting, audit logging

---

## 📚 FURTHER LEARNING

- **Database Books**: "Designing Data-Intensive Applications" by Martin Kleppmann
- **REST API Standards**: OpenAPI Specification, JSON:API, HAL
- **Performance**: Database indexing, query optimization guides
- **Security**: OWASP API Security Top 10
- **Monitoring**: Prometheus metrics, distributed tracing
