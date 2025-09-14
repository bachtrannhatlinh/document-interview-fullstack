# SQL vs NoSQL - INTERVIEW GUIDE

## 🗃️ SQL DATABASES (RELATIONAL)

### Characteristics
- **Structure**: Tables với rows và columns
- **Schema**: Fixed schema, cần define trước
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Relationships**: Foreign keys, JOINs
- **Query Language**: SQL (Structured Query Language)

### Popular SQL Databases
- **PostgreSQL**: Advanced features, JSON support, open source
- **MySQL**: Fast, widely used, good for web apps
- **SQLite**: Lightweight, embedded, good for development
- **SQL Server**: Microsoft, enterprise features

### Khi nào dùng SQL?
- Cần ACID compliance (banking, financial)
- Complex queries với nhiều JOINs
- Data có structure rõ ràng, ít thay đổi
- Reporting và analytics
- Cần consistency mạnh

---

## 📄 NoSQL DATABASES (NON-RELATIONAL)

### Types of NoSQL

#### 1. Document Databases (MongoDB, CouchDB)
```javascript
// MongoDB document
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "NYC"
  },
  hobbies: ["reading", "coding"]
}
```

#### 2. Key-Value Stores (Redis, DynamoDB)
```javascript
// Redis example
SET user:123 "{'name': 'John', 'email': 'john@example.com'}"
GET user:123
```

#### 3. Column-Family (Cassandra, HBase)
```
// Wide column structure
RowKey | Column1 | Column2 | Column3
user1  | name    | email   | age
```

#### 4. Graph Databases (Neo4j, Amazon Neptune)
```cypher
// Neo4j query
CREATE (john:Person {name: 'John'})
CREATE (jane:Person {name: 'Jane'})  
CREATE (john)-[:FRIENDS_WITH]->(jane)
```

### Khi nào dùng NoSQL?
- Cần scale horizontally (big data)
- Schema thay đổi thường xuyên
- Rapid development, agile projects
- Caching và session storage
- Real-time applications

---

## ⚖️ SO SÁNH CHI TIẾT

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (scale up) | Horizontal (scale out) |
| **ACID** | Full ACID support | Eventually consistent |
| **Queries** | Complex SQL queries | Simple queries |
| **Performance** | Good for complex queries | Fast for simple operations |
| **Maturity** | Very mature, decades old | Newer, rapidly evolving |

---

## 🎯 INTERVIEW QUESTIONS & ANSWERS

### 1. "Khi nào bạn chọn SQL vs NoSQL?"

**Answer Framework:**
```
Tôi sẽ chọn dựa trên:

SQL khi:
- Cần ACID compliance (financial data)
- Complex relationships giữa entities
- Structured data, schema ít thay đổi
- Cần complex queries, reporting
- Team đã familiar với SQL

NoSQL khi:  
- Cần scale nhanh, handle big data
- Schema thay đổi thường xuyên
- Rapid prototyping
- Caching layer
- Document-based data
```

### 2. "Giải thích ACID properties?"

```
A - Atomicity: Transaction all-or-nothing
C - Consistency: Data integrity rules
I - Isolation: Concurrent transactions không affect nhau  
D - Durability: Committed data persist sau crash

Ví dụ: Chuyển tiền
- A: Trừ tiền A và cộng tiền B cùng thành công/fail
- C: Tổng tiền không đổi
- I: Transaction khác không thấy intermediate state
- D: Sau khi commit, data không mất
```

### 3. "NoSQL có ACID không?"

```
- Hầu hết NoSQL: Eventually consistent
- Một số như MongoDB: Có ACID cho single document
- DynamoDB: ACID cho transactions
- Trade-off: Consistency vs Availability/Partition tolerance (CAP theorem)
```

### 4. "Denormalization trong NoSQL?"

```
SQL: Normalize để tránh duplication
NoSQL: Denormalize để optimize read performance

Example - User và Posts:
SQL: 2 tables với foreign key
NoSQL: Embed posts trong user document

Trade-off:
✅ Faster reads (1 query)  
❌ Data duplication
❌ Update complexity
```

---

## 💻 CODE EXAMPLES

### SQL với Node.js (PostgreSQL + Prisma)

```javascript
// schema.prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  content  String?
  authorId Int
  author   User   @relation(fields: [authorId], references: [id])
}

// JavaScript code
const user = await prisma.user.create({
  data: {
    name: 'John',
    email: 'john@example.com',
    posts: {
      create: [
        { title: 'First Post', content: 'Hello world' }
      ]
    }
  },
  include: {
    posts: true
  }
});
```

### NoSQL với Node.js (MongoDB + Mongoose)

```javascript
// Schema definition
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  posts: [{
    title: String,
    content: String,
    createdAt: { type: Date, default: Date.now }
  }]
});

// Usage
const user = new User({
  name: 'John',
  email: 'john@example.com',
  posts: [
    { title: 'First Post', content: 'Hello world' }
  ]
});

await user.save();

// Query
const users = await User.find({ 
  'posts.title': { $regex: /node/i } 
});
```

---

## 🚀 ADVANCED TOPICS

### Database Design Patterns

#### 1. SQL Patterns
- **Normalization**: 1NF, 2NF, 3NF
- **Indexing**: B-tree, Hash, Partial indexes
- **Partitioning**: Horizontal/Vertical splitting

#### 2. NoSQL Patterns  
- **Embedding vs Referencing**
- **Bucket Pattern**: Time-series data
- **Polymorphic Pattern**: Different schemas in same collection

### Scaling Strategies

#### SQL Scaling
```javascript
// Master-Slave replication
const masterDB = new Pool({ host: 'master-db' });
const slaveDB = new Pool({ host: 'slave-db' });

// Read from slave, write to master
const readQuery = (sql) => slaveDB.query(sql);
const writeQuery = (sql) => masterDB.query(sql);
```

#### NoSQL Scaling
```javascript
// MongoDB Sharding
// Auto-scales horizontally
db.users.createIndex({ userId: "hashed" });
sh.shardCollection("myapp.users", { userId: "hashed" });
```

---

## 📊 PERFORMANCE CONSIDERATIONS

### SQL Optimization
- **Proper indexing**
- **Query optimization** 
- **Connection pooling**
- **Prepared statements**

### NoSQL Optimization
- **Document design**
- **Appropriate data types**
- **Index usage**
- **Aggregation pipeline optimization**

---

## 🎯 SYSTEM DESIGN SCENARIOS

### Scenario 1: E-commerce Platform
```
Products Catalog: NoSQL (MongoDB)
- Flexible product attributes
- Fast reads for browsing

Orders & Payments: SQL (PostgreSQL)  
- ACID transactions
- Complex financial reporting

User Sessions: Redis (Key-Value)
- Fast access
- TTL support
```

### Scenario 2: Social Media App
```
User Profiles: NoSQL (MongoDB)
- Flexible user data
- Embedded posts/comments

Analytics: SQL (PostgreSQL)
- Complex aggregations  
- Time-series analysis

Real-time Chat: NoSQL (MongoDB) + Redis
- Document-based messages
- Redis for presence/caching
```

---

## ✅ CHECKLIST CHUẨN BỊ

**SQL Topics:**
- [ ] ACID properties
- [ ] Normalization vs Denormalization  
- [ ] Indexing strategies
- [ ] Transaction isolation levels
- [ ] SQL injection prevention

**NoSQL Topics:**
- [ ] Document vs Key-Value vs Graph
- [ ] Eventual consistency
- [ ] Sharding và replication
- [ ] CAP theorem
- [ ] Data modeling patterns

**Practical Skills:**
- [ ] Viết complex SQL queries
- [ ] MongoDB aggregation pipeline
- [ ] Database connection pooling
- [ ] Migration scripts
- [ ] Performance monitoring
