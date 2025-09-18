# System Design Interview Questions - Middle Level

## 1. Thiết kế hệ thống Chat Real-time

### Yêu cầu cơ bản
- Chat 1-on-1 và group chat
- Message real-time
- Online status
- Message history
- Scale đến vài triệu user

### Kiến trúc tổng quan

#### Frontend
```javascript
// WebSocket connection
const socket = new WebSocket('wss://chat-api.example.com');

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  displayMessage(message);
};

const sendMessage = (text, chatId) => {
  socket.send(JSON.stringify({
    type: 'message',
    chatId,
    text,
    timestamp: Date.now()
  }));
};
```

#### Backend Architecture
```
Load Balancer
    ↓
API Gateway
    ↓
WebSocket Servers (multiple instances)
    ↓
Message Queue (Redis/RabbitMQ)
    ↓
Message Service → Database
```

#### Database Schema
```sql
-- Messages table
CREATE TABLE messages (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  chat_id BIGINT NOT NULL,
  sender_id BIGINT NOT NULL,
  content TEXT NOT NULL,
  message_type ENUM('text', 'image', 'file'),
  created_at TIMESTAMP,
  INDEX idx_chat_id_created_at (chat_id, created_at)
);

-- Chats table
CREATE TABLE chats (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  type ENUM('direct', 'group'),
  name VARCHAR(255),
  created_at TIMESTAMP
);

-- Chat participants
CREATE TABLE chat_participants (
  chat_id BIGINT,
  user_id BIGINT,
  joined_at TIMESTAMP,
  PRIMARY KEY (chat_id, user_id)
);
```

#### WebSocket Server (Node.js)
```javascript
const WebSocket = require('ws');
const Redis = require('redis');

class ChatServer {
  constructor() {
    this.wss = new WebSocket.Server({ port: 8080 });
    this.redis = Redis.createClient();
    this.connections = new Map(); // userId -> WebSocket
    
    this.setupWebSocketHandlers();
    this.setupRedisSubscription();
  }
  
  setupWebSocketHandlers() {
    this.wss.on('connection', (ws) => {
      ws.on('message', async (data) => {
        const message = JSON.parse(data);
        
        switch (message.type) {
          case 'join':
            this.connections.set(message.userId, ws);
            this.updateOnlineStatus(message.userId, true);
            break;
            
          case 'message':
            await this.handleNewMessage(message);
            break;
        }
      });
      
      ws.on('close', () => {
        // Remove connection and update offline status
      });
    });
  }
  
  async handleNewMessage(message) {
    // Lưu message vào database
    const savedMessage = await this.saveMessage(message);
    
    // Publish to Redis cho các server khác
    await this.redis.publish('chat:message', JSON.stringify(savedMessage));
    
    // Gửi đến các participant trong chat
    const participants = await this.getChatParticipants(message.chatId);
    participants.forEach(userId => {
      const connection = this.connections.get(userId);
      if (connection) {
        connection.send(JSON.stringify(savedMessage));
      }
    });
  }
}
```

### Scaling Strategy
1. **Horizontal scaling**: Multiple WebSocket servers behind load balancer
2. **Database sharding**: Shard messages by chat_id
3. **Caching**: Redis cache cho recent messages, online users
4. **CDN**: Cho file uploads (images, documents)

### Challenges và Solutions
- **Connection persistence**: Sticky sessions hoặc Redis pub/sub
- **Message ordering**: Timestamp + message sequence
- **Offline users**: Message queue, push notifications

---

## 2. Thiết kế hệ thống Booking vé

### Yêu cầu
- Book vé máy bay/tàu/xe
- Tránh overbooking
- Handle high concurrency
- Payment processing
- Seat selection

### Database Design
```sql
-- Events (flights, shows, etc.)
CREATE TABLE events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255),
  total_seats INT,
  available_seats INT,
  event_date DATETIME,
  price DECIMAL(10,2)
);

-- Seats
CREATE TABLE seats (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  event_id BIGINT,
  seat_number VARCHAR(10),
  status ENUM('available', 'reserved', 'booked'),
  reserved_until DATETIME NULL,
  INDEX idx_event_status (event_id, status)
);

-- Bookings
CREATE TABLE bookings (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  event_id BIGINT,
  user_id BIGINT,
  seat_ids JSON,
  status ENUM('pending', 'confirmed', 'cancelled'),
  expires_at DATETIME,
  created_at TIMESTAMP
);
```

### Concurrency Control Strategies

#### 1. Pessimistic Locking
```javascript
// Service layer
class BookingService {
  async reserveSeats(eventId, seatNumbers, userId) {
    const transaction = await db.transaction();
    
    try {
      // Lock seats with SELECT FOR UPDATE
      const seats = await db.query(
        `SELECT * FROM seats 
         WHERE event_id = ? AND seat_number IN (?) 
         AND status = 'available' 
         FOR UPDATE`,
        [eventId, seatNumbers]
      );
      
      if (seats.length !== seatNumbers.length) {
        throw new Error('Some seats are not available');
      }
      
      // Reserve seats for 10 minutes
      await db.query(
        `UPDATE seats SET status = 'reserved', 
         reserved_until = DATE_ADD(NOW(), INTERVAL 10 MINUTE)
         WHERE id IN (?)`,
        [seats.map(s => s.id)]
      );
      
      // Create booking record
      const booking = await this.createBooking(eventId, userId, seats);
      
      await transaction.commit();
      return booking;
      
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }
}
```

#### 2. Optimistic Locking với Version
```sql
-- Thêm version column
ALTER TABLE seats ADD COLUMN version INT DEFAULT 0;

-- Update with version check
UPDATE seats 
SET status = 'reserved', 
    version = version + 1,
    reserved_until = DATE_ADD(NOW(), INTERVAL 10 MINUTE)
WHERE id = ? AND version = ? AND status = 'available';
```

#### 3. Redis Distributed Lock
```javascript
const Redis = require('redis');
const client = Redis.createClient();

class BookingService {
  async reserveSeatsWithLock(eventId, seatNumbers, userId) {
    const lockKey = `booking:${eventId}:${seatNumbers.join(',')}`;
    const lockValue = `${userId}:${Date.now()}`;
    
    // Acquire distributed lock
    const acquired = await client.set(lockKey, lockValue, 'PX', 10000, 'NX');
    
    if (!acquired) {
      throw new Error('Seats are being processed by another request');
    }
    
    try {
      return await this.processReservation(eventId, seatNumbers, userId);
    } finally {
      // Release lock
      await client.del(lockKey);
    }
  }
}
```

### Queue System cho High Traffic
```javascript
// Bull queue
const Queue = require('bull');
const bookingQueue = new Queue('seat booking', {
  redis: { port: 6379, host: '127.0.0.1' }
});

// Producer
app.post('/api/book', async (req, res) => {
  const job = await bookingQueue.add('process-booking', {
    userId: req.user.id,
    eventId: req.body.eventId,
    seatNumbers: req.body.seats
  });
  
  res.json({ bookingId: job.id, status: 'processing' });
});

// Consumer
bookingQueue.process('process-booking', async (job) => {
  return await bookingService.reserveSeats(
    job.data.eventId,
    job.data.seatNumbers,
    job.data.userId
  );
});
```

### Cleanup Job cho Expired Reservations
```javascript
// Cron job chạy mỗi phút
const cron = require('node-cron');

cron.schedule('* * * * *', async () => {
  await db.query(`
    UPDATE seats 
    SET status = 'available', reserved_until = NULL 
    WHERE status = 'reserved' AND reserved_until < NOW()
  `);
});
```

---

## 3. API Search với Autocomplete

### Yêu cầu
- Tìm kiếm sản phẩm/địa điểm
- Autocomplete real-time
- Typo tolerance
- Fast response (<100ms)
- Support multiple languages

### Architecture Overview
```
User Input → Frontend → API Gateway → Search Service → Elasticsearch
                                   ↘ Cache Layer (Redis)
```

### Elasticsearch Index Structure
```javascript
// Product index mapping
const productMapping = {
  mappings: {
    properties: {
      id: { type: 'keyword' },
      name: {
        type: 'text',
        analyzer: 'standard',
        fields: {
          suggest: {
            type: 'completion',
            analyzer: 'simple'
          },
          keyword: {
            type: 'keyword'
          }
        }
      },
      category: { type: 'keyword' },
      description: { type: 'text' },
      price: { type: 'double' },
      tags: { type: 'keyword' },
      created_at: { type: 'date' }
    }
  }
};
```

### API Implementation
```javascript
const { Client } = require('@elastic/elasticsearch');

class SearchService {
  constructor() {
    this.client = new Client({ node: 'http://localhost:9200' });
    this.redis = require('redis').createClient();
  }
  
  async autocomplete(query, limit = 10) {
    // Check cache first
    const cacheKey = `autocomplete:${query}`;
    const cached = await this.redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    const body = {
      suggest: {
        product_suggest: {
          prefix: query,
          completion: {
            field: 'name.suggest',
            size: limit,
            skip_duplicates: true
          }
        }
      }
    };
    
    const response = await this.client.search({
      index: 'products',
      body
    });
    
    const suggestions = response.body.suggest.product_suggest[0].options.map(
      option => ({
        text: option.text,
        score: option._score,
        source: option._source
      })
    );
    
    // Cache for 5 minutes
    await this.redis.setex(cacheKey, 300, JSON.stringify(suggestions));
    
    return suggestions;
  }
  
  async search(query, filters = {}, page = 1, size = 20) {
    const body = {
      query: {
        bool: {
          must: [
            {
              multi_match: {
                query: query,
                fields: ['name^2', 'description', 'tags'],
                fuzziness: 'AUTO'
              }
            }
          ],
          filter: []
        }
      },
      highlight: {
        fields: {
          name: {},
          description: {}
        }
      },
      from: (page - 1) * size,
      size: size
    };
    
    // Add filters
    if (filters.category) {
      body.query.bool.filter.push({
        term: { category: filters.category }
      });
    }
    
    if (filters.priceRange) {
      body.query.bool.filter.push({
        range: {
          price: {
            gte: filters.priceRange.min,
            lte: filters.priceRange.max
          }
        }
      });
    }
    
    const response = await this.client.search({
      index: 'products',
      body
    });
    
    return {
      hits: response.body.hits.hits,
      total: response.body.hits.total.value,
      took: response.body.took
    };
  }
}
```

### Frontend Implementation
```javascript
// Debounced autocomplete
import { debounce } from 'lodash';

class AutocompleteComponent {
  constructor() {
    this.debouncedSearch = debounce(this.search.bind(this), 300);
  }
  
  async search(query) {
    if (query.length < 2) return;
    
    try {
      const response = await fetch(`/api/search/autocomplete?q=${query}`);
      const suggestions = await response.json();
      this.displaySuggestions(suggestions);
    } catch (error) {
      console.error('Search error:', error);
    }
  }
  
  handleInput(event) {
    const query = event.target.value;
    this.debouncedSearch(query);
  }
  
  displaySuggestions(suggestions) {
    const container = document.getElementById('suggestions');
    container.innerHTML = suggestions.map(item => `
      <div class="suggestion-item" onclick="this.selectItem('${item.text}')">
        <strong>${item.text}</strong>
        <span>${item.source.category}</span>
      </div>
    `).join('');
  }
}
```

### Performance Optimizations
1. **Caching**: Redis cho popular queries
2. **Index warming**: Pre-load frequent searches
3. **Sharding**: Distribute data across multiple nodes
4. **Edge caching**: CDN cho static suggestions

---

## 4. Phân tích Performance Bottleneck

### Methodology: TOP-DOWN Approach

#### 1. Monitoring Stack
```javascript
// Application Performance Monitoring
const newrelic = require('newrelic');
const winston = require('winston');

// Custom metrics
const promClient = require('prom-client');
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status']
});

app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.url, res.statusCode)
      .observe(duration);
  });
  
  next();
});
```

#### 2. Frontend Performance Analysis

##### Phương pháp phân tích:
```javascript
// Core Web Vitals monitoring
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'largest-contentful-paint') {
      console.log('LCP:', entry.startTime);
    }
    if (entry.entryType === 'first-input') {
      console.log('FID:', entry.processingStart - entry.startTime);
    }
    if (entry.entryType === 'layout-shift') {
      console.log('CLS:', entry.value);
    }
  }
});

observer.observe({entryTypes: ['largest-contentful-paint', 'first-input', 'layout-shift']});

// Bundle analysis
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false
    })
  ]
};
```

##### Common Issues & Solutions:
- **Large bundle size**: Code splitting, lazy loading
- **Render blocking**: Critical CSS inlining
- **Memory leaks**: Event listener cleanup
- **Re-renders**: React.memo, useMemo, useCallback

#### 3. Backend Performance Analysis

##### Request tracing:
```javascript
const express = require('express');
const app = express();

// Request tracing middleware
app.use((req, res, next) => {
  req.startTime = Date.now();
  req.traceId = generateTraceId();
  
  const originalSend = res.send;
  res.send = function(body) {
    const duration = Date.now() - req.startTime;
    
    logger.info('Request completed', {
      traceId: req.traceId,
      method: req.method,
      url: req.url,
      duration,
      statusCode: res.statusCode
    });
    
    originalSend.call(this, body);
  };
  
  next();
});

// Database query monitoring
const mysql = require('mysql2');
const pool = mysql.createPool({
  // config
});

const originalQuery = pool.query;
pool.query = function(sql, params, callback) {
  const start = Date.now();
  
  return originalQuery.call(this, sql, params, (error, results) => {
    const duration = Date.now() - start;
    
    if (duration > 1000) { // Log slow queries
      logger.warn('Slow query detected', {
        sql: sql,
        duration,
        params
      });
    }
    
    callback(error, results);
  });
};
```

##### Performance Profiling:
```javascript
// CPU profiling
const v8Profiler = require('v8-profiler-next');

function startCPUProfiling() {
  v8Profiler.startProfiling('cpu-profile', true);
  
  setTimeout(() => {
    const profile = v8Profiler.stopProfiling('cpu-profile');
    profile.export(function(error, result) {
      fs.writeFileSync('cpu-profile.cpuprofile', result);
    });
  }, 30000);
}

// Memory profiling
function takeHeapSnapshot() {
  const snapshot = v8Profiler.takeSnapshot();
  snapshot.export(function(error, result) {
    fs.writeFileSync('heap-snapshot.heapsnapshot', result);
  });
}
```

#### 4. Database Performance Analysis

##### Query Analysis:
```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1;

-- Analyze query execution
EXPLAIN ANALYZE SELECT * FROM orders 
WHERE customer_id = 123 AND created_at > '2024-01-01';

-- Check index usage
SHOW INDEX FROM orders;

-- Monitor query performance
SELECT 
  query,
  total_time / 1000000 as total_time_ms,
  calls,
  mean_time / 1000000 as avg_time_ms
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;
```

##### Index Optimization:
```sql
-- Composite index for common query patterns
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);

-- Covering index
CREATE INDEX idx_orders_summary ON orders(customer_id) INCLUDE (total_amount, status);

-- Partial index for common conditions
CREATE INDEX idx_active_orders ON orders(created_at) WHERE status = 'active';
```

#### 5. Network Analysis

##### API Response Time Monitoring:
```javascript
// Response time middleware
app.use((req, res, next) => {
  const start = process.hrtime();
  
  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start);
    const duration = seconds * 1000 + nanoseconds / 1000000;
    
    // Alert on slow responses
    if (duration > 5000) {
      alert.send(`Slow API response: ${req.url} took ${duration}ms`);
    }
  });
  
  next();
});

// External service monitoring
const axios = require('axios');

const client = axios.create({
  timeout: 5000
});

client.interceptors.request.use(config => {
  config.metadata = { startTime: Date.now() };
  return config;
});

client.interceptors.response.use(
  response => {
    const duration = Date.now() - response.config.metadata.startTime;
    logger.info('External API call', {
      url: response.config.url,
      duration,
      status: response.status
    });
    return response;
  },
  error => {
    const duration = Date.now() - error.config.metadata.startTime;
    logger.error('External API error', {
      url: error.config.url,
      duration,
      error: error.message
    });
    throw error;
  }
);
```

### Systematic Debugging Process

1. **Establish baseline metrics**
2. **Identify the bottleneck layer** (Frontend/Backend/Database/Network)
3. **Drill down into specific components**
4. **Use profiling tools** for deep analysis
5. **Implement fixes incrementally**
6. **Measure improvement** and validate

### Tools & Commands
```bash
# System monitoring
top
htop
iostat
vmstat
netstat -an
ss -tulpn

# Application monitoring
node --prof app.js
node --prof-process isolate-*.log

# Database monitoring
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS;

# Network monitoring
curl -w "@curl-format.txt" -o /dev/null -s "http://example.com"
```

---

## Tổng kết

Những điểm quan trọng cần nhớ:

1. **Scalability**: Luôn design cho growth
2. **Consistency vs Availability**: Hiểu trade-offs
3. **Monitoring**: Implement từ đầu, không phải sau
4. **Caching**: Multiple levels (browser, CDN, application, database)
5. **Security**: Authentication, authorization, input validation
6. **Testing**: Unit, integration, load testing
7. **Documentation**: API docs, system architecture

Khi phỏng vấn, hãy:
- Hỏi clarifying questions
- Bắt đầu từ high-level design
- Discuss trade-offs
- Đề cập monitoring & scaling
- Nói về error handling
