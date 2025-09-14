# CÂU HỎI PHỎNG VẤN NODE.JS

## 🔥 CÂU HỎI CƠ BẢN

### 1. **Event Loop**
**Q: Giải thích Event Loop trong Node.js hoạt động như thế nào?**

**A:** Event Loop là core của Node.js, cho phép xử lý bất đồng bộ:
- **Call Stack**: Thực thi code đồng bộ
- **Callback Queue**: Chứa callbacks từ Web APIs
- **Microtask Queue**: Chứa Promises và process.nextTick()
- **Event Loop**: Kiểm tra Call Stack rỗng → chuyển callbacks từ Queue vào Stack

**Thứ tự ưu tiên:**
1. Call Stack (đồng bộ)
2. Microtask Queue (Promises, process.nextTick)
3. Callback Queue (setTimeout, setInterval)

---

### 2. **Asynchronous Programming**
**Q: Sự khác biệt giữa Callback, Promise và Async/Await?**

**A:**
- **Callback**: Function được truyền vào function khác, gọi khi hoàn thành
- **Promise**: Object đại diện cho kết quả tương lai (pending, fulfilled, rejected)
- **Async/Await**: Syntax sugar cho Promise, code dễ đọc hơn

**Ví dụ:**
```javascript
// Callback Hell
getData(function(a) {
    getMoreData(a, function(b) {
        getEvenMoreData(b, function(c) {
            // callback hell
        });
    });
});

// Promise
getData()
    .then(a => getMoreData(a))
    .then(b => getEvenMoreData(b))
    .catch(error => console.log(error));

// Async/Await
async function fetchData() {
    try {
        const a = await getData();
        const b = await getMoreData(a);
        const c = await getEvenMoreData(b);
        return c;
    } catch (error) {
        console.log(error);
    }
}
```

---

### 3. **Module System**
**Q: CommonJS vs ES6 Modules?**

**A:**
- **CommonJS** (require/module.exports):
  - Đồng bộ, load tại runtime
  - Dynamic imports
  - Node.js default

- **ES6 Modules** (import/export):
  - Bất đồng bộ, load tại compile time
  - Static imports
  - Tree shaking support

**Ví dụ:**
```javascript
// CommonJS
const fs = require('fs');
module.exports = { myFunction };

// ES6 Modules
import fs from 'fs';
export { myFunction };
```

---

## 🚀 CÂU HỎI NÂNG CAO

### 4. **Memory Management**
**Q: Làm thế nào để tránh memory leaks trong Node.js?**

**A:**
- **Closures**: Tránh giữ reference không cần thiết
- **Event Listeners**: Remove listeners khi không dùng
- **Timers**: Clear intervals/timeouts
- **Global Variables**: Tránh tạo global variables
- **Streams**: Properly close streams

**Ví dụ:**
```javascript
// Memory leak
setInterval(() => {
    const data = new Array(1000000).fill('data');
    // data không được garbage collected
}, 1000);

// Fix
let intervalId = setInterval(() => {
    const data = new Array(1000000).fill('data');
}, 1000);

// Clear khi cần
clearInterval(intervalId);
```

---

### 5. **Error Handling**
**Q: Cách xử lý lỗi trong Node.js application?**

**A:**
- **Try-catch**: Cho đồng bộ code
- **Promise.catch()**: Cho Promise chains
- **process.on('uncaughtException')**: Global error handler
- **process.on('unhandledRejection')**: Unhandled Promise rejections
- **Domain module**: Isolate errors (deprecated)

**Ví dụ:**
```javascript
// Global error handlers
process.on('uncaughtException', (err) => {
    console.error('Uncaught Exception:', err);
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

// Async error handling
async function riskyOperation() {
    try {
        const result = await someAsyncOperation();
        return result;
    } catch (error) {
        console.error('Error in riskyOperation:', error);
        throw error; // Re-throw nếu cần
    }
}
```

---

### 6. **Performance Optimization**
**Q: Cách optimize performance Node.js application?**

**A:**
- **Clustering**: Sử dụng tất cả CPU cores
- **Caching**: Redis, Memcached
- **Streaming**: Xử lý data theo chunks
- **Compression**: Gzip, Brotli
- **Database optimization**: Indexing, connection pooling
- **Monitoring**: Profiling, metrics

**Ví dụ Clustering:**
```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork(); // Restart worker
    });
} else {
    // Worker process
    require('./app.js');
    console.log(`Worker ${process.pid} started`);
}
```

---

## 🏗️ CÂU HỎI SYSTEM DESIGN

### 7. **Microservices vs Monolith**
**Q: Khi nào nên dùng Microservices?**

**A:**
**Microservices phù hợp khi:**
- Team lớn (50+ developers)
- Cần scale độc lập các services
- Có domain boundaries rõ ràng
- Cần deploy độc lập

**Monolith phù hợp khi:**
- Team nhỏ
- Application đơn giản
- Cần development nhanh
- Chưa có kinh nghiệm với distributed systems

---

### 8. **Database Design**
**Q: Thiết kế database cho e-commerce system?**

**A:**
**Tables chính:**
- Users (id, email, password_hash, created_at)
- Products (id, name, price, description, stock)
- Orders (id, user_id, total, status, created_at)
- Order_Items (id, order_id, product_id, quantity, price)
- Categories (id, name, parent_id)

**Indexes:**
- Users: email (unique)
- Products: name, category_id
- Orders: user_id, created_at
- Order_Items: order_id, product_id

---

## 💻 CODING CHALLENGES

### 9. **File Processing**
**Q: Tạo API đọc file CSV và trả về JSON**

```javascript
const fs = require('fs');
const csv = require('csv-parser');
const { Transform } = require('stream');

function csvToJson(filePath) {
    return new Promise((resolve, reject) => {
        const results = [];
        
        fs.createReadStream(filePath)
            .pipe(csv())
            .on('data', (data) => results.push(data))
            .on('end', () => resolve(results))
            .on('error', reject);
    });
}

// Usage
csvToJson('data.csv')
    .then(data => console.log(JSON.stringify(data, null, 2)))
    .catch(console.error);
```

---

### 10. **Rate Limiting**
**Q: Implement rate limiting middleware**

```javascript
const rateLimit = require('express-rate-limit');

// Basic rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});

// Advanced with Redis
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const client = redis.createClient();

const redisLimiter = rateLimit({
    store: new RedisStore({
        client: client,
        prefix: 'rl:'
    }),
    windowMs: 15 * 60 * 1000,
    max: 100
});

app.use('/api/', redisLimiter);
```

---

## 🎯 CÂU HỎI HÀNH VI

### 11. **Problem Solving**
**Q: Kể về lần bạn debug một bug khó?**

**A:** Structure trả lời:
1. **Situation**: Mô tả context và vấn đề
2. **Task**: Nhiệm vụ cần làm
3. **Action**: Các bước đã thực hiện
4. **Result**: Kết quả đạt được

**Ví dụ:**
- Bug: Memory leak trong production
- Action: Used profiling tools, identified closure issue
- Result: Fixed memory leak, improved performance by 40%

---

### 12. **Learning & Growth**
**Q: Cách bạn stay updated với Node.js?**

**A:**
- Follow Node.js official blog
- Read GitHub releases
- Join communities (Node.js Discord, Reddit)
- Contribute to open source
- Attend conferences (NodeConf, JSConf)
- Practice coding challenges

---

## 📝 TIPS TRẢ LỜI

1. **Be specific**: Đưa ra ví dụ cụ thể
2. **Think out loud**: Giải thích thought process
3. **Ask questions**: Clarify requirements
4. **Admit when unsure**: Thành thật khi không biết
5. **Show enthusiasm**: Thể hiện passion với technology

---

**Chúc bạn phỏng vấn thành công! 🚀**
