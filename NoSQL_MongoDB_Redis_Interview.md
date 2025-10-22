# NoSQL - MONGODB & REDIS CHO PHỎNG VẤN

## MỤC LỤC
1. [NoSQL là gì - Tại sao cần NoSQL](#nosql-overview)
2. [MongoDB - Document Database](#mongodb)
3. [Redis - In-Memory Cache](#redis)
4. [SQL vs NoSQL - Khi nào dùng](#sql-vs-nosql)
5. [Câu hỏi phỏng vấn](#interview-questions)

---

## NoSQL OVERVIEW

### NoSQL là gì?

**NoSQL = "Not Only SQL"**
- Không bắt buộc schema cố định (flexible schema)
- Horizontal scaling dễ dàng hơn
- Phù hợp với unstructured/semi-structured data
- High performance cho specific use cases

### Các loại NoSQL:

| Type | Example | Use Case |
|------|---------|----------|
| **Document** | MongoDB, CouchDB | General purpose, JSON data |
| **Key-Value** | Redis, DynamoDB | Caching, session storage |
| **Column-Family** | Cassandra, HBase | Time-series, analytics |
| **Graph** | Neo4j | Social networks, recommendations |

---

## MONGODB - DOCUMENT DATABASE

### 1. MongoDB là gì?

MongoDB lưu data dưới dạng **documents** (JSON-like format gọi là BSON)

**So sánh SQL vs MongoDB:**
```
SQL:
- Database → Tables → Rows → Columns
- Fixed schema

MongoDB:
- Database → Collections → Documents → Fields
- Flexible schema
```

**Ví dụ:**
```javascript
// SQL Table: Users
UserId | UserName | Email              | Age
-------|----------|--------------------|----|
1      | John     | john@email.com     | 25 |

// MongoDB Document: users collection
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "userName": "John",
  "email": "john@email.com",
  "age": 25,
  "address": {  // Nested object - SQL cần bảng riêng!
    "city": "Hanoi",
    "street": "Ba Dinh"
  },
  "hobbies": ["reading", "gaming"]  // Array - SQL khó lưu!
}
```

---

### 2. CRUD Operations - MongoDB

#### Insert Documents
```javascript
// Insert 1 document
db.users.insertOne({
  userName: "John",
  email: "john@email.com",
  age: 25,
  city: "Hanoi"
});

// Insert nhiều documents
db.users.insertMany([
  { userName: "Alice", email: "alice@email.com", age: 30 },
  { userName: "Bob", email: "bob@email.com", age: 22 }
]);
```

#### Find (Query) Documents
```javascript
// Tìm tất cả
db.users.find();

// Tìm với điều kiện
db.users.find({ city: "Hanoi" });

// Tìm với nhiều điều kiện
db.users.find({ 
  city: "Hanoi", 
  age: { $gt: 20 }  // age > 20
});

// Tìm 1 document
db.users.findOne({ email: "john@email.com" });

// Projection: Chỉ lấy fields cần thiết
db.users.find(
  { city: "Hanoi" },
  { userName: 1, email: 1, _id: 0 }  // 1 = include, 0 = exclude
);

// Sort
db.users.find().sort({ age: -1 });  // -1 = DESC, 1 = ASC

// Limit & Skip (Pagination)
db.users.find().skip(10).limit(5);  // Page 3, 5 per page
```

#### Update Documents
```javascript
// Update 1 document
db.users.updateOne(
  { email: "john@email.com" },  // Filter
  { $set: { age: 26 } }          // Update
);

// Update nhiều documents
db.users.updateMany(
  { city: "Hanoi" },
  { $set: { country: "Vietnam" } }
);

// Update hoặc Insert (Upsert)
db.users.updateOne(
  { email: "new@email.com" },
  { $set: { userName: "NewUser" } },
  { upsert: true }  // Insert nếu không tìm thấy
);

// Increment value
db.users.updateOne(
  { email: "john@email.com" },
  { $inc: { age: 1 } }  // age = age + 1
);

// Add to array
db.users.updateOne(
  { email: "john@email.com" },
  { $push: { hobbies: "swimming" } }
);
```

#### Delete Documents
```javascript
// Delete 1 document
db.users.deleteOne({ email: "john@email.com" });

// Delete nhiều documents
db.users.deleteMany({ city: "Hanoi" });

// Delete tất cả
db.users.deleteMany({});
```

---

### 3. Query Operators

```javascript
// Comparison Operators
db.users.find({ age: { $eq: 25 } });    // age = 25
db.users.find({ age: { $gt: 25 } });    // age > 25
db.users.find({ age: { $gte: 25 } });   // age >= 25
db.users.find({ age: { $lt: 25 } });    // age < 25
db.users.find({ age: { $lte: 25 } });   // age <= 25
db.users.find({ age: { $ne: 25 } });    // age != 25

// IN operator
db.users.find({ city: { $in: ["Hanoi", "HCMC"] } });

// Logical Operators
db.users.find({
  $and: [
    { age: { $gt: 20 } },
    { city: "Hanoi" }
  ]
});

db.users.find({
  $or: [
    { city: "Hanoi" },
    { city: "HCMC" }
  ]
});

// Exists
db.users.find({ phone: { $exists: true } });  // Có field phone

// Regex
db.users.find({ email: { $regex: /gmail\.com$/ } });  // Email gmail
```

---

### 4. Aggregation Pipeline (Quan trọng!)

**Aggregation = GROUP BY trong SQL**

```javascript
// Đếm users theo city (giống GROUP BY)
db.users.aggregate([
  {
    $group: {
      _id: "$city",              // GROUP BY city
      count: { $sum: 1 },        // COUNT(*)
      avgAge: { $avg: "$age" }   // AVG(age)
    }
  }
]);

/*
Result:
[
  { _id: "Hanoi", count: 5, avgAge: 27 },
  { _id: "HCMC", count: 3, avgAge: 30 }
]
*/

// Pipeline với nhiều stages
db.orders.aggregate([
  // Stage 1: Match (WHERE)
  { $match: { status: "completed" } },
  
  // Stage 2: Group (GROUP BY)
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      orderCount: { $sum: 1 }
    }
  },
  
  // Stage 3: Sort (ORDER BY)
  { $sort: { totalSpent: -1 } },
  
  // Stage 4: Limit (LIMIT)
  { $limit: 10 }
]);

// Lookup (JOIN trong MongoDB)
db.orders.aggregate([
  {
    $lookup: {
      from: "users",              // Bảng join
      localField: "customerId",   // Foreign key
      foreignField: "_id",        // Primary key
      as: "customerInfo"          // Output field
    }
  }
]);
```

**So sánh SQL vs MongoDB Aggregation:**
```javascript
// SQL:
SELECT city, COUNT(*), AVG(age)
FROM users
WHERE age > 18
GROUP BY city
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC
LIMIT 10;

// MongoDB:
db.users.aggregate([
  { $match: { age: { $gt: 18 } } },          // WHERE
  {
    $group: {                                 // GROUP BY
      _id: "$city",
      count: { $sum: 1 },
      avgAge: { $avg: "$age" }
    }
  },
  { $match: { count: { $gt: 5 } } },         // HAVING
  { $sort: { count: -1 } },                  // ORDER BY
  { $limit: 10 }                              // LIMIT
]);
```

---

### 5. Indexes trong MongoDB

```javascript
// Tạo index
db.users.createIndex({ email: 1 });  // 1 = ascending

// Compound index
db.users.createIndex({ city: 1, age: -1 });

// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Xem indexes
db.users.getIndexes();

// Drop index
db.users.dropIndex("email_1");

// Explain query (như Execution Plan)
db.users.find({ email: "john@email.com" }).explain("executionStats");
```

---

### 6. Schema Design - Best Practices

#### Embedded Documents (Nested)
```javascript
// GOOD: Dùng khi data liên quan chặt chẽ và READ cùng nhau
{
  _id: 1,
  userName: "John",
  address: {  // Embedded
    street: "123 Main St",
    city: "Hanoi",
    zipCode: "10000"
  },
  orders: [  // Array of embedded documents
    { orderId: 101, product: "Laptop", price: 1000 },
    { orderId: 102, product: "Mouse", price: 20 }
  ]
}

// Use case:
// - 1 user có 1 address (one-to-one)
// - User ít orders (one-to-few)
// - Luôn query user + orders cùng nhau
```

#### References (Normalized)
```javascript
// GOOD: Dùng khi data độc lập hoặc quan hệ many-to-many

// Users collection
{
  _id: ObjectId("user1"),
  userName: "John"
}

// Orders collection
{
  _id: ObjectId("order1"),
  customerId: ObjectId("user1"),  // Reference
  product: "Laptop",
  price: 1000
}

// Use case:
// - 1 user có nhiều orders (one-to-many)
// - Orders được query độc lập
// - Tránh document quá lớn
```

**Nguyên tắc:**
- **Embed** khi: Data liên quan chặt, read cùng nhau, ít update
- **Reference** khi: Data độc lập, many-to-many, document có thể lớn

---

### 7. Transactions trong MongoDB

```javascript
// MongoDB 4.0+ hỗ trợ multi-document transactions
const session = db.getMongo().startSession();
session.startTransaction();

try {
  const usersCollection = session.getDatabase("mydb").users;
  const ordersCollection = session.getDatabase("mydb").orders;
  
  // Update user
  usersCollection.updateOne(
    { _id: ObjectId("user1") },
    { $inc: { balance: -100 } },
    { session }
  );
  
  // Insert order
  ordersCollection.insertOne(
    { userId: ObjectId("user1"), amount: 100 },
    { session }
  );
  
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
} finally {
  session.endSession();
}
```

---

## REDIS - IN-MEMORY CACHE

### 1. Redis là gì?

**Redis = Remote Dictionary Server**
- In-memory key-value store (nhanh vô cùng!)
- Data structure server (không chỉ strings)
- Thường dùng cho caching, session storage, pub/sub

**Đặc điểm:**
- Tất cả data trong RAM → Cực nhanh (< 1ms)
- Persistent options: RDB snapshots, AOF logs
- Single-threaded nhưng vẫn rất nhanh

---

### 2. Data Types trong Redis

#### String (Key-Value)
```bash
# Set key-value
SET user:1:name "John"
SET user:1:age 25

# Get value
GET user:1:name  # "John"

# Multiple set/get
MSET user:2:name "Alice" user:2:age 30
MGET user:1:name user:2:name  # ["John", "Alice"]

# Increment
SET counter 10
INCR counter  # 11
INCRBY counter 5  # 16

# Set với expiration (TTL)
SETEX session:abc123 3600 "user_data"  # Expire sau 3600 giây (1 giờ)

# Set nếu chưa tồn tại
SETNX lock:resource "locked"  # Only set nếu key chưa có
```

#### Hash (Object/Dictionary)
```bash
# Set hash fields (giống object trong JS)
HSET user:1 name "John" age 25 email "john@email.com"

# Get field
HGET user:1 name  # "John"

# Get all fields
HGETALL user:1  
# ["name", "John", "age", "25", "email", "john@email.com"]

# Increment hash field
HINCRBY user:1 age 1  # age = 26
```

#### List (Array)
```bash
# Push to list
LPUSH notifications "New message"  # Push to left
RPUSH notifications "New order"    # Push to right

# Pop from list
LPOP notifications  # Get & remove from left
RPOP notifications  # Get & remove from right

# Get range
LRANGE notifications 0 9  # Get first 10 items

# List length
LLEN notifications
```

#### Set (Unique values)
```bash
# Add to set
SADD tags:post1 "nodejs" "javascript" "backend"

# Check if member exists
SISMEMBER tags:post1 "nodejs"  # 1 (true)

# Get all members
SMEMBERS tags:post1  # ["nodejs", "javascript", "backend"]

# Set operations
SADD tags:post2 "javascript" "react" "frontend"
SINTER tags:post1 tags:post2  # Intersection: ["javascript"]
SUNION tags:post1 tags:post2  # Union: all unique tags
```

#### Sorted Set (Set với score)
```bash
# Add to sorted set
ZADD leaderboard 100 "Alice" 85 "Bob" 95 "Charlie"

# Get top players (highest scores)
ZREVRANGE leaderboard 0 2 WITHSCORES
# ["Alice", "100", "Charlie", "95", "Bob", "85"]

# Get rank
ZRANK leaderboard "Bob"  # 0 (lowest rank)
ZREVRANK leaderboard "Bob"  # 2 (từ cao xuống thấp)

# Increment score
ZINCRBY leaderboard 10 "Bob"  # Bob's score = 95
```

---

### 3. Redis Use Cases

#### Use Case 1: Caching
```javascript
// Node.js + Redis caching example
const redis = require('redis');
const client = redis.createClient();

async function getUser(userId) {
  // 1. Check cache first
  const cacheKey = `user:${userId}`;
  const cached = await client.get(cacheKey);
  
  if (cached) {
    console.log('Cache HIT');
    return JSON.parse(cached);
  }
  
  // 2. Cache MISS → Query database
  console.log('Cache MISS');
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  
  // 3. Store in cache (expire after 1 hour)
  await client.setEx(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}
```

**Cache Strategies:**
- **Cache-Aside**: Application quản lý cache (pattern trên)
- **Write-Through**: Ghi vào cache + DB cùng lúc
- **Write-Behind**: Ghi vào cache trước, background sync DB

#### Use Case 2: Session Storage
```javascript
// Express.js session với Redis
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: 'your-secret',
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 3600000 } // 1 hour
}));

// Session được lưu trong Redis với TTL tự động
```

#### Use Case 3: Rate Limiting
```javascript
// Rate limit: 100 requests per hour
async function checkRateLimit(userId) {
  const key = `rate_limit:${userId}`;
  const current = await client.incr(key);
  
  if (current === 1) {
    // First request, set expiration
    await client.expire(key, 3600); // 1 hour
  }
  
  if (current > 100) {
    throw new Error('Rate limit exceeded');
  }
  
  return true;
}
```

#### Use Case 4: Leaderboard
```javascript
// Gaming leaderboard
async function updateScore(playerId, score) {
  await client.zAdd('leaderboard', {
    score: score,
    value: playerId
  });
}

async function getTopPlayers(limit = 10) {
  return await client.zRangeWithScores('leaderboard', 0, limit - 1, {
    REV: true // Descending order
  });
}
```

#### Use Case 5: Pub/Sub (Real-time messaging)
```javascript
// Publisher
await client.publish('notifications', JSON.stringify({
  type: 'new_order',
  orderId: 12345
}));

// Subscriber
const subscriber = client.duplicate();
await subscriber.connect();

subscriber.subscribe('notifications', (message) => {
  const data = JSON.parse(message);
  console.log('Received:', data);
  // Handle notification
});
```

---

### 4. Redis Persistence

```bash
# RDB (Snapshot)
# - Định kỳ save toàn bộ data vào file
# - Nhanh, compact
# - Có thể mất data giữa 2 snapshots

SAVE     # Blocking save
BGSAVE   # Background save

# Config trong redis.conf:
save 900 1    # Save nếu có 1 change trong 900 giây
save 300 10   # Save nếu có 10 changes trong 300 giây

# AOF (Append Only File)
# - Log mọi write operation
# - Durable hơn
# - File lớn hơn

# Config:
appendonly yes
appendfsync everysec  # Sync every second (balance giữa performance & safety)
```

---

### 5. Redis Best Practices

```bash
# 1. Naming convention cho keys
user:1000:profile
user:1000:sessions
order:5000:details

# 2. Set TTL để tránh memory đầy
EXPIRE user:1000:session 3600

# 3. Use pipelines cho multiple commands
# Giảm network round-trips
MULTI
SET key1 "value1"
SET key2 "value2"
SET key3 "value3"
EXEC

# 4. Monitor memory usage
INFO memory
MEMORY USAGE user:1000

# 5. Avoid blocking operations trong production
# ❌ KEYS * (scan toàn bộ DB)
# ✅ SCAN (iterate từng phần)
```

---

## SQL vs NoSQL - KHI NÀO DÙNG

### So sánh tổng quan:

| Tiêu chí | SQL | NoSQL |
|----------|-----|-------|
| **Schema** | Fixed, structured | Flexible, dynamic |
| **Scaling** | Vertical (scale up) | Horizontal (scale out) |
| **Transactions** | ACID đảm bảo | Eventually consistent |
| **Relationships** | JOINs mạnh mẽ | Denormalized, embeds |
| **Use Cases** | Financial, ERP | Social media, IoT, caching |

### Khi nào dùng SQL:
✅ Data có cấu trúc rõ ràng, quan hệ phức tạp
✅ Cần ACID transactions (banking, e-commerce)
✅ Complex queries với nhiều JOINs
✅ Data ít thay đổi schema
✅ Business intelligence, reporting

### Khi nào dùng MongoDB:
✅ Flexible schema (startup, MVP)
✅ Hierarchical data (JSON, nested objects)
✅ Horizontal scaling (millions of users)
✅ Fast development, iterate quickly
✅ Content management, catalogs, user profiles

### Khi nào dùng Redis:
✅ Caching (tăng tốc database queries)
✅ Session management
✅ Real-time analytics (leaderboards, counters)
✅ Pub/Sub messaging
✅ Rate limiting

### Pattern thực tế (Hybrid):
```
PostgreSQL/MySQL: Main database (persistent, structured)
       ↓
   MongoDB: Flexible data, logs, analytics
       ↓
    Redis: Cache layer, sessions, real-time features

Example:
- User profiles → PostgreSQL (structured, relationships)
- User activity logs → MongoDB (flexible, high volume)
- Session data → Redis (fast access, TTL)
```

---

## INTERVIEW QUESTIONS

### Câu 1: "So sánh SQL và NoSQL, khi nào dùng từng loại?"

**Trả lời:**
```
SQL (PostgreSQL, MySQL):
✅ Data có cấu trúc, schema cố định
✅ Cần ACID transactions (chuyển tiền, đặt hàng)
✅ Complex queries với JOINs
VD: Banking, ERP systems

NoSQL (MongoDB):
✅ Flexible schema (prototype nhanh)
✅ Hierarchical/nested data
✅ Horizontal scaling
VD: Social media, IoT, content management

Thực tế: Dùng cả 2!
- SQL: Core business data
- NoSQL: Logs, caching, analytics
```

### Câu 2: "Giải thích caching strategy với Redis?"

**Trả lời:**
```javascript
// Cache-Aside Pattern (Lazy Loading)
async function getData(key) {
  // 1. Check cache
  let data = await redis.get(key);
  if (data) return JSON.parse(data);
  
  // 2. Cache miss → Query DB
  data = await database.query(key);
  
  // 3. Store in cache
  await redis.setEx(key, 3600, JSON.stringify(data));
  
  return data;
}

Benefits:
- Reduce DB load
- Fast response time
- Scale reads easily

Trade-offs:
- Stale data (giải quyết bằng TTL)
- Cache invalidation complexity
```

### Câu 3: "MongoDB aggregation pipeline là gì?"

**Trả lời:**
```javascript
// Aggregation = Xử lý data qua nhiều stages
db.orders.aggregate([
  { $match: { status: "completed" } },     // WHERE
  { $group: {                               // GROUP BY
      _id: "$customerId",
      total: { $sum: "$amount" }
    }
  },
  { $sort: { total: -1 } },                // ORDER BY
  { $limit: 10 }                            // LIMIT
]);

// Giống như SQL nhưng xử lý document-oriented data
// Pipeline: Mỗi stage xử lý output của stage trước
```

### Câu 4: "Embedded vs References trong MongoDB?"

**Trả lời:**
```javascript
// EMBEDDED: One-to-few, data thường query cùng nhau
{
  _id: 1,
  name: "John",
  addresses: [  // Embed
    { city: "Hanoi", street: "..." }
  ]
}

// REFERENCE: One-to-many, data query độc lập
// Users collection
{ _id: 1, name: "John" }

// Orders collection (nhiều orders)
{ _id: 101, userId: 1, amount: 100 }

Rule of thumb:
- Embed: < 100 sub-documents, read together
- Reference: > 100, independent queries
```

### Câu 5: "Redis persistence: RDB vs AOF?"

**Trả lời:**
```
RDB (Snapshot):
✅ Fast, compact file
✅ Good cho backups
❌ Có thể mất data giữa 2 snapshots
Use: Backup hàng ngày, non-critical data

AOF (Append Only File):
✅ More durable, log mọi operation
✅ Có thể replay để recover
❌ File lớn hơn, slower restart
Use: Critical data, cần durability cao

Thực tế: Dùng cả 2!
- RDB: Backup định kỳ
- AOF: Continuous persistence
```

### Câu 6: "N+1 problem trong MongoDB?"

**Trả lời:**
```javascript
// ❌ N+1 Problem
const users = await db.users.find();
for (let user of users) {  // N queries
  const orders = await db.orders.find({ userId: user._id });
}

// ✅ Solution 1: Aggregation $lookup (JOIN)
db.users.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  }
]);

// ✅ Solution 2: Embed orders (nếu phù hợp)
{
  _id: 1,
  name: "John",
  orders: [...]  // Embedded
}
```

### Câu 7: "Indexing trong MongoDB?"

**Trả lời:**
```javascript
// Tạo index để tăng tốc queries
db.users.createIndex({ email: 1 });  // Single field
db.users.createIndex({ city: 1, age: -1 });  // Compound

// Giống SQL:
✅ Faster queries (đặc biệt WHERE, sort)
❌ Slower writes (phải update index)
❌ More storage

Best practices:
- Index fields trong queries thường xuyên
- Compound index: thứ tự quan trọng
- Avoid over-indexing
```

### Câu 8: "Redis atomic operations?"

**Trả lời:**
```bash
# Redis single-threaded → All operations atomic!

# Example: Inventory management
WATCH product:123:stock  # Watch for changes
stock = GET product:123:stock

if stock > 0:
  MULTI
    DECR product:123:stock
    # Add to cart...
  EXEC
else:
  UNWATCH
  # Out of stock

# WATCH-MULTI-EXEC = Optimistic locking
# Rollback nếu key bị modify bởi client khác
```

### Câu 9: "Scaling strategies: SQL vs NoSQL?"

**Trả lời:**
```
SQL (Vertical Scaling - Scale Up):
- Tăng CPU, RAM, Disk
- Read replicas cho reads
- Sharding phức tạp
Limit: Hardware có giới hạn

NoSQL (Horizontal Scaling - Scale Out):
- Thêm nhiều servers
- Built-in sharding (MongoDB)
- Auto-balancing
Unlimited: Cứ thêm server

Trade-off:
SQL: Strong consistency, complex queries
NoSQL: Eventual consistency, simple queries, scale ∞
```

### Câu 10: "Cache invalidation strategies?"

**Trả lời:**
```javascript
// Problem: "Cache invalidation là 1 trong 2 hardest problems!"

// Strategy 1: TTL (Time To Live)
await redis.setEx(key, 3600, data);  // Expire 1h
// ✅ Simple
// ❌ Stale data trong TTL period

// Strategy 2: Write-through
async function updateUser(id, data) {
  await db.update(id, data);
  await redis.set(`user:${id}`, JSON.stringify(data));
}
// ✅ Cache always fresh
// ❌ Slower writes

// Strategy 3: Event-driven invalidation
// Khi update DB → Publish event → Invalidate cache
await db.update(id, data);
await redis.del(`user:${id}`);
// ✅ Cache stays fresh
// ✅ Faster reads (no write-through overhead)

Thực tế: Combine strategies!
- TTL cho safety
- Explicit invalidation cho critical data
```

---

## NODEJS + MONGODB + REDIS CODE EXAMPLES

### Setup
```bash
npm install mongodb redis
```

### MongoDB Connection
```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017');

async function main() {
  await client.connect();
  const db = client.db('myapp');
  const users = db.collection('users');
  
  // CRUD operations
  await users.insertOne({ name: 'John', age: 25 });
  const user = await users.findOne({ name: 'John' });
  await users.updateOne({ name: 'John' }, { $set: { age: 26 } });
  await users.deleteOne({ name: 'John' });
  
  await client.close();
}
```

### Redis Connection
```javascript
const redis = require('redis');

const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

await client.connect();

// String operations
await client.set('user:1:name', 'John');
const name = await client.get('user:1:name');

// Hash operations
await client.hSet('user:1', {
  name: 'John',
  age: '25',
  email: 'john@email.com'
});
const user = await client.hGetAll('user:1');

await client.disconnect();
```

### Real-world: API với caching
```javascript
const express = require('express');
const { MongoClient } = require('mongodb');
const redis = require('redis');

const app = express();
const mongoClient = new MongoClient('mongodb://localhost:27017');
const redisClient = redis.createClient();

await mongoClient.connect();
await redisClient.connect();

const db = mongoClient.db('myapp');

app.get('/users/:id', async (req, res) => {
  const userId = req.params.id;
  const cacheKey = `user:${userId}`;
  
  // 1. Check cache
  const cached = await redisClient.get(cacheKey);
  if (cached) {
    console.log('Cache HIT');
    return res.json(JSON.parse(cached));
  }
  
  // 2. Query MongoDB
  console.log('Cache MISS');
  const user = await db.collection('users').findOne({ _id: userId });
  
  // 3. Cache result (1 hour)
  await redisClient.setEx(cacheKey, 3600, JSON.stringify(user));
  
  res.json(user);
});

app.listen(3000);
```

---

## CHECKLIST CHUẨN BỊ PHỎNG VẤN

### MongoDB:
- [ ] CRUD operations (insert, find, update, delete)
- [ ] Query operators ($gt, $in, $regex, etc.)
- [ ] Aggregation pipeline ($match, $group, $lookup)
- [ ] Indexing
- [ ] Embedded vs References
- [ ] Schema design principles

### Redis:
- [ ] Data types (String, Hash, List, Set, Sorted Set)
- [ ] Basic commands (GET, SET, HSET, LPUSH, ZADD)
- [ ] TTL & expiration
- [ ] Common use cases (caching, sessions, leaderboard)
- [ ] Persistence (RDB vs AOF)

### General:
- [ ] SQL vs NoSQL comparison
- [ ] When to use each database type
- [ ] CAP theorem (basic understanding)
- [ ] Scaling strategies
- [ ] Caching strategies

---

## TÀI NGUYÊN HỌC TẬP

### MongoDB:
- Official Docs: https://docs.mongodb.com/
- MongoDB University (Free courses): https://university.mongodb.com/
- Playground: https://mongoplayground.net/

### Redis:
- Official Docs: https://redis.io/documentation
- Try Redis: https://try.redis.io/
- Redis University (Free): https://university.redis.com/

### Practice:
- MongoDB CRUD: Tạo blog app (posts, comments, users)
- Redis: Implement rate limiting, leaderboard
- Combined: API với caching layer

---

**GOOD LUCK! 🚀**

NoSQL không khó, chỉ là approach khác SQL. Hiểu use cases là quan trọng nhất!
