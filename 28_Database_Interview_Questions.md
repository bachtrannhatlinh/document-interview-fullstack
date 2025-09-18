# 🔹 Database Interview Questions & Answers

## 1. So sánh SQL và NoSQL. Khi nào nên dùng PostgreSQL, khi nào MongoDB?

### SQL vs NoSQL

| **Tiêu chí** | **SQL (PostgreSQL)** | **NoSQL (MongoDB)** |
|--------------|---------------------|---------------------|
| **Schema** | Cố định, strict schema | Linh hoạt, schema-less |
| **Cấu trúc** | Bảng, hàng, cột | Document, key-value, graph |
| **ACID** | Hỗ trợ đầy đủ | Hỗ trợ có giới hạn |
| **Scaling** | Vertical scaling | Horizontal scaling |
| **Query** | SQL chuẩn | Syntax riêng |
| **Consistency** | Strong consistency | Eventual consistency |

### Khi nào dùng PostgreSQL?
- **Dữ liệu có cấu trúc rõ ràng**: Banking, finance, e-commerce
- **Cần ACID transactions**: Payment processing, inventory management
- **Complex queries**: Reports, analytics, joins phức tạp
- **Data integrity quan trọng**: User accounts, orders
- **Established team**: Team đã quen với SQL

**Ví dụ use cases:**
```sql
-- Banking transaction
BEGIN;
UPDATE accounts SET balance = balance - 1000 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE user_id = 2;
COMMIT;
```

### Khi nào dùng MongoDB?
- **Dữ liệu không có cấu trúc cố định**: Content management, logs
- **Rapid development**: Prototype, startup MVP
- **Horizontal scaling**: High traffic applications
- **JSON-like data**: APIs, web applications
- **Real-time features**: Chat, notifications

**Ví dụ use cases:**
```javascript
// User profile với nested data
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  preferences: {
    theme: "dark",
    notifications: {
      email: true,
      push: false
    }
  },
  tags: ["developer", "nodejs", "react"]
}
```

## 2. Transaction và ACID. Tại sao quan trọng?

### Transaction là gì?
Transaction là một nhóm các operations được thực thi như một đơn vị duy nhất. Tất cả đều thành công hoặc tất cả đều thất bại.

### ACID Properties

#### **A - Atomicity (Tính nguyên tử)**
- Tất cả operations trong transaction đều thành công hoặc tất cả đều rollback
- Không có trạng thái "một phần hoàn thành"

```sql
-- Ví dụ: Transfer money
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- Cả hai đều thành công hoặc cả hai đều rollback
```

#### **C - Consistency (Tính nhất quán)**
- Database luôn ở trạng thái hợp lệ trước và sau transaction
- Constraints, triggers được respect

```sql
-- Balance không được âm (constraint)
ALTER TABLE accounts ADD CONSTRAINT check_balance CHECK (balance >= 0);
```

#### **I - Isolation (Tính cô lập)**
- Các transactions không ảnh hưởng lẫn nhau
- Isolation levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable

```sql
-- Transaction A không thấy changes chưa commit của Transaction B
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

#### **D - Durability (Tính bền vững)**
- Sau khi commit, changes được lưu vĩnh viễn
- Survive system crashes, power failures

### Tại sao ACID quan trọng?
- **Data integrity**: Đảm bảo data luôn đúng
- **Reliability**: System hoạt động đúng dù có lỗi
- **Concurrency**: Nhiều users có thể work cùng lúc
- **Recovery**: Có thể recover sau crashes

## 3. Index là gì? Khi nào index giúp nhanh hơn, khi nào lại gây chậm?

### Index là gì?
Index là cấu trúc dữ liệu giúp tăng tốc độ truy vấn bằng cách tạo "shortcuts" đến data.

### Các loại Index

#### **B-Tree Index** (Phổ biến nhất)
```sql
CREATE INDEX idx_user_email ON users(email);
-- Good for: =, <, >, BETWEEN, ORDER BY
```

#### **Hash Index**
```sql
CREATE INDEX idx_user_id USING HASH ON users(id);
-- Good for: = only, rất nhanh
```

#### **Partial Index**
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
-- Chỉ index những records cần thiết
```

#### **Composite Index**
```sql
CREATE INDEX idx_user_status_created ON users(status, created_at);
-- For multi-column queries
```

### Khi nào Index giúp nhanh hơn?

#### **SELECT Queries**
```sql
-- Với index on email
SELECT * FROM users WHERE email = 'john@example.com';
-- O(log n) thay vì O(n)
```

#### **ORDER BY**
```sql
-- Với index on created_at
SELECT * FROM posts ORDER BY created_at DESC;
-- Không cần sort, đã sorted sẵn trong index
```

#### **JOIN Operations**
```sql
-- Index on foreign keys
SELECT u.name, p.title 
FROM users u 
JOIN posts p ON u.id = p.user_id;
```

#### **WHERE với conditions phổ biến**
```sql
-- Index on status
SELECT * FROM orders WHERE status = 'pending';
```

### Khi nào Index gây chậm?

#### **INSERT/UPDATE/DELETE Operations**
```sql
-- Mỗi INSERT phải update index
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
-- Phải update index trên name và email
```

#### **Too Many Indexes**
```sql
-- 10 indexes trên 1 table = 10 structures cần maintain
CREATE INDEX idx1 ON users(name);
CREATE INDEX idx2 ON users(email);
-- ... 8 indexes khác
-- Mỗi INSERT sẽ rất chậm
```

#### **Large Data Modifications**
```sql
-- Update toàn bộ table
UPDATE users SET status = 'inactive' WHERE created_at < '2020-01-01';
-- Tất cả indexes liên quan đều phải rebuild
```

#### **Wrong Index Usage**
```sql
-- Index on (status, created_at) nhưng query:
SELECT * FROM users WHERE created_at > '2023-01-01' AND status = 'active';
-- Sai thứ tự columns trong WHERE
```

### Best Practices cho Index
```sql
-- 1. Index các columns hay dùng trong WHERE
CREATE INDEX idx_user_email ON users(email);

-- 2. Index foreign keys
CREATE INDEX idx_post_user_id ON posts(user_id);

-- 3. Composite index theo thứ tự selectivity
CREATE INDEX idx_user_status_created ON users(status, created_at);
-- status có ít distinct values, created_at có nhiều

-- 4. Partial index cho common conditions
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

## 4. N+1 Query Problem là gì? Làm sao để tránh?

### N+1 Problem là gì?
N+1 problem xảy ra khi:
- 1 query để lấy danh sách records
- N queries để lấy related data cho mỗi record

### Ví dụ N+1 Problem

#### **Bad Example:**
```javascript
// 1 query để lấy users
const users = await User.findAll(); // 1 query

// N queries để lấy posts của mỗi user
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } }); // N queries
}
// Total: 1 + N queries
```

#### **SQL Generated:**
```sql
-- Query 1
SELECT * FROM users;

-- Query 2 (for user 1)
SELECT * FROM posts WHERE user_id = 1;

-- Query 3 (for user 2)
SELECT * FROM posts WHERE user_id = 2;

-- ... N more queries
```

### Cách tránh N+1 Problem

#### **1. Eager Loading (Include/Joins)**
```javascript
// Sequelize
const users = await User.findAll({
  include: [Post] // 1 query với JOIN
});

// Prisma
const users = await prisma.user.findMany({
  include: { posts: true }
});
```

**SQL Generated:**
```sql
SELECT u.*, p.* 
FROM users u 
LEFT JOIN posts p ON u.id = p.user_id;
```

#### **2. Batch Loading với IN clause**
```javascript
// Lấy tất cả users trước
const users = await User.findAll();
const userIds = users.map(u => u.id);

// 1 query để lấy tất cả posts
const posts = await Post.findAll({
  where: { userId: { $in: userIds } }
});

// Group posts theo userId trong memory
const postsByUserId = posts.reduce((acc, post) => {
  acc[post.userId] = acc[post.userId] || [];
  acc[post.userId].push(post);
  return acc;
}, {});

// Assign posts cho users
users.forEach(user => {
  user.posts = postsByUserId[user.id] || [];
});
```

#### **3. DataLoader Pattern** (GraphQL)
```javascript
const postLoader = new DataLoader(async (userIds) => {
  const posts = await Post.findAll({
    where: { userId: { $in: userIds } }
  });
  
  return userIds.map(userId => 
    posts.filter(post => post.userId === userId)
  );
});

// Usage
const posts1 = await postLoader.load(1);
const posts2 = await postLoader.load(2);
// Chỉ 1 database query được thực thi
```

#### **4. Query Optimization với Select Fields**
```javascript
// Chỉ select fields cần thiết
const users = await User.findAll({
  attributes: ['id', 'name', 'email'], // Không lấy fields không cần
  include: [{
    model: Post,
    attributes: ['id', 'title', 'user_id']
  }]
});
```

### Monitoring N+1 Problems

#### **Enable Query Logging**
```javascript
// Sequelize
const sequelize = new Sequelize(database, username, password, {
  logging: console.log, // Log tất cả SQL queries
});

// Prisma
// Thêm vào schema.prisma
generator client {
  provider = "prisma-client-js"
  log = ["query", "info", "warn", "error"]
}
```

#### **Use Profiling Tools**
```javascript
// Express middleware để count queries
app.use((req, res, next) => {
  let queryCount = 0;
  
  const originalQuery = sequelize.query;
  sequelize.query = function(...args) {
    queryCount++;
    return originalQuery.apply(this, args);
  };
  
  res.on('finish', () => {
    console.log(`${req.method} ${req.path}: ${queryCount} queries`);
  });
  
  next();
});
```

## 5. Optimistic vs Pessimistic Locking: khác nhau thế nào?

### Pessimistic Locking (Khóa bi quan)

#### **Cách hoạt động:**
- Lock data ngay khi đọc
- Giữ lock cho đến khi transaction kết thúc
- Ngăn không cho transactions khác access data

#### **Implementation:**
```sql
-- SQL Server / PostgreSQL
BEGIN;
SELECT * FROM products WHERE id = 1 FOR UPDATE;
-- Product bị lock, transactions khác phải chờ
UPDATE products SET quantity = quantity - 1 WHERE id = 1;
COMMIT; -- Lock được release
```

```javascript
// Sequelize
const transaction = await sequelize.transaction();
const product = await Product.findByPk(1, {
  lock: transaction.LOCK.UPDATE,
  transaction
});

product.quantity -= 1;
await product.save({ transaction });
await transaction.commit();
```

#### **Ưu điểm:**
- **Guaranteed consistency**: Không có race conditions
- **Simple logic**: Không cần handle conflicts
- **Suitable for**: High contention scenarios

#### **Nhược điểm:**
- **Performance impact**: Blocks other transactions
- **Deadlock risk**: Transactions có thể block lẫn nhau
- **Scalability issues**: Reduced concurrency

### Optimistic Locking (Khóa lạc quan)

#### **Cách hoạt động:**
- Không lock data khi đọc
- Check conflicts khi update
- Retry nếu có conflicts

#### **Implementation với Version Field:**
```sql
-- Add version column
ALTER TABLE products ADD version INTEGER DEFAULT 0;

-- Read data
SELECT id, name, quantity, version FROM products WHERE id = 1;
-- version = 5

-- Update với version check
UPDATE products 
SET quantity = quantity - 1, version = version + 1
WHERE id = 1 AND version = 5;

-- Nếu affected rows = 0, có conflict xảy ra
```

#### **Implementation với Timestamp:**
```sql
-- Using updated_at
UPDATE products 
SET quantity = quantity - 1, updated_at = NOW()
WHERE id = 1 AND updated_at = '2023-12-01 10:30:00';
```

#### **Implementation trong code:**
```javascript
// Optimistic locking với retry
async function updateProductQuantity(productId, quantity) {
  const maxRetries = 3;
  
  for (let retry = 0; retry < maxRetries; retry++) {
    try {
      // Read current data
      const product = await Product.findByPk(productId);
      const originalVersion = product.version;
      
      // Modify data
      const updatedRows = await Product.update(
        { 
          quantity: product.quantity - quantity,
          version: originalVersion + 1
        },
        {
          where: {
            id: productId,
            version: originalVersion // Check version
          }
        }
      );
      
      if (updatedRows[0] === 0) {
        // Conflict detected, retry
        if (retry === maxRetries - 1) {
          throw new Error('Update failed after max retries');
        }
        continue;
      }
      
      return; // Success
      
    } catch (error) {
      if (retry === maxRetries - 1) throw error;
    }
  }
}
```

#### **Ưu điểm:**
- **Better performance**: No locking overhead
- **High concurrency**: Multiple reads simultaneously
- **No deadlocks**: Không có blocking
- **Scalability**: Better for distributed systems

#### **Nhược điểm:**
- **Complex logic**: Phải handle conflicts
- **Retry logic**: Cần implement retry mechanism
- **Starvation risk**: Một transaction có thể fail liên tục

### So sánh và When to Use

| **Aspect** | **Pessimistic** | **Optimistic** |
|------------|-----------------|----------------|
| **Performance** | Slower, blocking | Faster, non-blocking |
| **Concurrency** | Lower | Higher |
| **Complexity** | Simple | Complex |
| **Conflicts** | Prevented | Detected & handled |
| **Best for** | High contention | Low contention |

### Use Cases

#### **Pessimistic Locking:**
```javascript
// Banking transactions
async function transferMoney(fromAccount, toAccount, amount) {
  const transaction = await sequelize.transaction();
  
  // Lock both accounts
  const from = await Account.findByPk(fromAccount, {
    lock: transaction.LOCK.UPDATE,
    transaction
  });
  
  const to = await Account.findByPk(toAccount, {
    lock: transaction.LOCK.UPDATE,
    transaction
  });
  
  // Safe to modify
  from.balance -= amount;
  to.balance += amount;
  
  await Promise.all([
    from.save({ transaction }),
    to.save({ transaction })
  ]);
  
  await transaction.commit();
}
```

#### **Optimistic Locking:**
```javascript
// Inventory management
async function reserveProduct(productId, quantity) {
  const maxRetries = 5;
  
  for (let i = 0; i < maxRetries; i++) {
    const product = await Product.findByPk(productId);
    
    if (product.quantity < quantity) {
      throw new Error('Insufficient inventory');
    }
    
    const success = await Product.update(
      { 
        quantity: product.quantity - quantity,
        version: product.version + 1
      },
      {
        where: {
          id: productId,
          version: product.version
        }
      }
    );
    
    if (success[0] > 0) return; // Success
    
    // Retry with exponential backoff
    await new Promise(resolve => 
      setTimeout(resolve, Math.pow(2, i) * 100)
    );
  }
  
  throw new Error('Failed to reserve after retries');
}
```

### Hybrid Approach
```javascript
// Combine both strategies
async function updateCriticalData(id, data) {
  // Use optimistic for normal cases
  try {
    return await optimisticUpdate(id, data);
  } catch (conflictError) {
    // Fall back to pessimistic for high contention
    return await pessimisticUpdate(id, data);
  }
}
```

---

## 🎯 Tips for Interview Success

### Preparation Strategy:
1. **Practice SQL queries** - Joins, subqueries, window functions
2. **Understand trade-offs** - Khi nào dùng gì và tại sao
3. **Real-world examples** - Prepare examples từ projects đã làm
4. **Performance tuning** - Explain how to optimize slow queries
5. **Scaling strategies** - Replication, sharding, caching

### Common Follow-up Questions:
- "How would you handle this in a distributed system?"
- "What happens if the database goes down?"
- "How do you ensure data consistency across services?"
- "What monitoring would you put in place?"

---
*File này sẽ giúp bạn trả lời tự tin các câu hỏi database trong phỏng vấn fullstack developer!*
