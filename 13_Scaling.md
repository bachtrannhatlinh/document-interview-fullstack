# 13. Scaling

## Horizontal vs Vertical Scaling
- **Vertical Scaling (Scale Up)**: Tăng tài nguyên server hiện tại (CPU, RAM, Disk)
  - Ưu điểm: Đơn giản, không cần thay đổi architecture
  - Nhược điểm: Giới hạn phần cứng, single point of failure
  
- **Horizontal Scaling (Scale Out)**: Thêm nhiều server/instance
  - Ưu điểm: Không giới hạn, fault tolerance cao
  - Nhược điểm: Phức tạp, cần load balancer, data synchronization

## Node.js Scaling Techniques

### 1. Cluster Module
Node.js chạy đơn luồng, muốn tận dụng đa nhân CPU thì dùng module `cluster` để tạo nhiều process con (worker) chia sẻ port.

```js
const cluster = require('cluster');
const os = require('os');
const express = require('express');

if (cluster.isMaster) {
  // Tạo worker cho mỗi CPU core
  for (let i = 0; i < os.cpus().length; i++) {
    cluster.fork();
  }
  
  // Restart worker khi crash
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork();
  });
} else {
  // Worker process
  const app = express();
  app.get('/', (req, res) => {
    res.send(`Process ${process.pid} handling request`);
  });
  app.listen(3000);
}
```

### 2. Child Process
Tạo process con để chạy task nặng, tách biệt với main process.

```js
const { spawn, fork, exec } = require('child_process');

// Spawn - chạy command
const ls = spawn('ls', ['-lh', '/usr']);
ls.stdout.on('data', data => console.log(`stdout: ${data}`));

// Fork - tạo Node.js process con
const child = fork('./heavy-task.js');
child.send({ data: 'process this' });
child.on('message', (result) => console.log(result));

// Exec - chạy shell command
exec('ls -la', (error, stdout, stderr) => {
  if (error) throw error;
  console.log(stdout);
});
```

### 3. Worker Threads
Tạo thread con để xử lý tính toán nặng mà không block event loop (Node.js >= v10.5).

```js
// main.js
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  // Main thread
  const worker = new Worker(__filename);
  worker.postMessage({ numbers: [1, 2, 3, 4, 5] });
  worker.on('message', (result) => {
    console.log('Result:', result);
  });
} else {
  // Worker thread
  parentPort.on('message', ({ numbers }) => {
    const sum = numbers.reduce((a, b) => a + b, 0);
    parentPort.postMessage(sum);
  });
}
```

## Load Balancing Strategies

### 1. Round Robin
```js
// Simple round robin implementation
class RoundRobin {
  constructor(servers) {
    this.servers = servers;
    this.current = 0;
  }
  
  getNext() {
    const server = this.servers[this.current];
    this.current = (this.current + 1) % this.servers.length;
    return server;
  }
}
```

### 2. Nginx Configuration
```nginx
upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### 3. PM2 Cluster Mode
```bash
# pm2.config.js
module.exports = {
  apps: [{
    name: 'app',
    script: './app.js',
    instances: 'max', // hoặc số cụ thể
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production'
    }
  }]
};

pm2 start pm2.config.js
```

## Caching Strategies

### 1. In-Memory Cache
```js
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

app.get('/api/data/:id', async (req, res) => {
  const { id } = req.params;
  const cacheKey = `data_${id}`;
  
  // Check cache first
  let data = cache.get(cacheKey);
  if (!data) {
    // Fetch from database
    data = await database.findById(id);
    cache.set(cacheKey, data);
  }
  
  res.json(data);
});
```

### 2. Redis Cache
```js
const redis = require('redis');
const client = redis.createClient();

app.get('/api/user/:id', async (req, res) => {
  const { id } = req.params;
  
  try {
    // Check Redis cache
    const cached = await client.get(`user:${id}`);
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Fetch from database
    const user = await User.findById(id);
    
    // Cache for 1 hour
    await client.setex(`user:${id}`, 3600, JSON.stringify(user));
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Database Scaling

### 1. Read Replicas
```js
const mysql = require('mysql2');

const masterDB = mysql.createConnection({
  host: 'master-db-host',
  user: 'root',
  password: 'password',
  database: 'myapp'
});

const slaveDB = mysql.createConnection({
  host: 'slave-db-host',
  user: 'root',
  password: 'password',
  database: 'myapp'
});

// Write operations
const createUser = (userData) => {
  return masterDB.execute('INSERT INTO users SET ?', userData);
};

// Read operations
const getUsers = () => {
  return slaveDB.execute('SELECT * FROM users');
};
```

### 2. Database Sharding
```js
class DatabaseSharding {
  constructor(shards) {
    this.shards = shards;
  }
  
  getShard(key) {
    const hash = this.hashFunction(key);
    const shardIndex = hash % this.shards.length;
    return this.shards[shardIndex];
  }
  
  hashFunction(key) {
    let hash = 0;
    for (let i = 0; i < key.length; i++) {
      hash = ((hash << 5) - hash) + key.charCodeAt(i);
      hash = hash & hash; // Convert to 32bit integer
    }
    return Math.abs(hash);
  }
}
```

## Auto Scaling với Docker & Kubernetes

### Docker Compose Scale
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000-3005:3000"
    environment:
      - NODE_ENV=production
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
```

```bash
docker-compose up --scale app=5
```

### Kubernetes HPA (Horizontal Pod Autoscaler)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Monitoring & Performance

### 1. Health Check Endpoint
```js
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    pid: process.pid
  });
});
```

### 2. Rate Limiting
```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

app.use('/api/', limiter);
```

### 3. Circuit Breaker Pattern
```js
class CircuitBreaker {
  constructor(threshold = 5, timeout = 10000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.failureCount = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}
```

## Best Practices cho Scaling

1. **Stateless Design**: Không lưu state trong application server
2. **Database Connection Pooling**: Tái sử dụng connection
3. **Async Programming**: Sử dụng async/await, Promise
4. **Caching**: Cache ở nhiều tầng (memory, Redis, CDN)
5. **Resource Cleanup**: Đóng connection, clear timer
6. **Graceful Shutdown**: Handle SIGTERM signal
7. **Environment Variables**: Config cho từng environment

## Câu hỏi phỏng vấn thường gặp

1. **So sánh cluster vs worker_threads?**
   - Cluster: Tạo process con, chia sẻ port, tốt cho I/O intensive
   - Worker threads: Tạo thread con, shared memory, tốt cho CPU intensive

2. **Khi nào nên scale horizontal vs vertical?**
   - Vertical: Ứng dụng nhỏ, đơn giản, cost thấp
   - Horizontal: High traffic, fault tolerance, unlimited scaling

3. **Làm sao handle session khi scale horizontal?**
   - Session store (Redis, MongoDB)
   - JWT tokens (stateless)
   - Sticky sessions (not recommended)

4. **Database scaling strategies?**
   - Read replicas cho read-heavy workload
   - Sharding cho write-heavy workload  
   - Caching để giảm database load
