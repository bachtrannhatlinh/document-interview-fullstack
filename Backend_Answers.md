# TRẢ LỜI CHI TIẾT - BACKEND (NODE.JS / EXPRESS)

## 🔹 BACKEND (NODE.JS / EXPRESS)

---

### **1. Middleware trong Express.js là gì? Có các loại middleware nào?**

**A: EXPRESS.JS MIDDLEWARE**

**Middleware** là các functions thực thi trong request-response cycle, có quyền truy cập `req`, `res` objects và `next()` function.

**Cú pháp cơ bản:**
```javascript
function middleware(req, res, next) {
    // Logic xử lý
    console.log('Time:', Date.now());
    next(); // Chuyển đến middleware tiếp theo
}

app.use(middleware);
```

**CÁC LOẠI MIDDLEWARE:**

**1. Application-level middleware:**
```javascript
const express = require('express');
const app = express();

// Middleware cho tất cả routes
app.use((req, res, next) => {
    console.log('Request URL:', req.originalUrl);
    next();
});

// Middleware cho specific route
app.use('/users', (req, res, next) => {
    console.log('Users route accessed');
    next();
});

// Multiple middleware functions
app.get('/protected', 
    authenticateToken,
    authorizeUser,
    (req, res) => {
        res.json({ message: 'Protected data' });
    }
);
```

**2. Router-level middleware:**
```javascript
const router = express.Router();

// Router middleware
router.use((req, res, next) => {
    console.log('Router Time:', Date.now());
    next();
});

// Specific router middleware
router.use('/users/:id', (req, res, next) => {
    console.log('Request Type:', req.method);
    next();
});

router.get('/users/:id', (req, res) => {
    res.json({ userId: req.params.id });
});

app.use('/api', router);
```

**3. Error-handling middleware:**
```javascript
// Error middleware (4 parameters)
app.use((err, req, res, next) => {
    console.error('Error:', err.stack);
    
    if (err.type === 'validation') {
        return res.status(400).json({
            error: 'Validation Error',
            details: err.message
        });
    }
    
    if (err.type === 'authentication') {
        return res.status(401).json({
            error: 'Authentication Required'
        });
    }
    
    // Default error
    res.status(500).json({
        error: 'Internal Server Error',
        message: process.env.NODE_ENV === 'development' ? err.message : 'Something went wrong'
    });
});

// Custom error throwing
app.get('/error-example', (req, res, next) => {
    const error = new Error('Custom error');
    error.type = 'validation';
    next(error); // Pass to error middleware
});
```

**4. Built-in middleware:**
```javascript
// Static files
app.use(express.static('public'));
app.use('/uploads', express.static('uploads'));

// JSON parsing
app.use(express.json({ limit: '10mb' }));

// URL-encoded parsing
app.use(express.urlencoded({ extended: true }));
```

**5. Third-party middleware:**
```javascript
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const compression = require('compression');
const rateLimit = require('express-rate-limit');

// Security headers
app.use(helmet());

// CORS handling
app.use(cors({
    origin: ['http://localhost:3000', 'https://mydomain.com'],
    credentials: true
}));

// Request logging
app.use(morgan('combined'));

// Response compression
app.use(compression());

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);
```

**Custom Authentication Middleware:**
```javascript
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid or expired token' });
        }
        
        req.user = user;
        next();
    });
}

function authorizeRole(roles) {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
}

// Usage
app.get('/admin', 
    authenticateToken,
    authorizeRole(['admin']),
    (req, res) => {
        res.json({ message: 'Admin area' });
    }
);
```

---

### **2. Phân biệt giữa cluster và worker threads trong Node.js**

**A: CLUSTER VS WORKER THREADS**

| **Cluster** | **Worker Threads** |
|-------------|-------------------|
| **Multi-process** | **Multi-thread** |
| Chia sẻ server port | Chia sẻ memory |
| Process isolation | Shared memory space |
| CPU-bound + I/O tasks | CPU-bound tasks |
| Higher memory usage | Lower memory usage |

**CLUSTER MODULE:**
```javascript
// master.js
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`Master ${process.pid} is running`);
    
    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    // Handle worker death
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        console.log('Starting a new worker');
        cluster.fork();
    });
    
    // Graceful shutdown
    process.on('SIGTERM', () => {
        console.log('Master shutting down');
        for (const id in cluster.workers) {
            cluster.workers[id].kill();
        }
    });
    
} else {
    // Worker process
    const express = require('express');
    const app = express();
    
    app.get('/', (req, res) => {
        res.json({
            message: 'Hello from worker',
            pid: process.pid
        });
    });
    
    app.get('/heavy', (req, res) => {
        // CPU-intensive task
        const start = Date.now();
        let counter = 0;
        while (Date.now() - start < 5000) {
            counter++;
        }
        
        res.json({
            message: 'Heavy task completed',
            pid: process.pid,
            counter
        });
    });
    
    const server = app.listen(3000, () => {
        console.log(`Worker ${process.pid} started`);
    });
    
    // Graceful worker shutdown
    process.on('SIGTERM', () => {
        console.log(`Worker ${process.pid} shutting down`);
        server.close(() => {
            process.exit(0);
        });
    });
}
```

**WORKER THREADS:**
```javascript
// main.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');
const path = require('path');

if (isMainThread) {
    // Main thread - Express server
    const express = require('express');
    const app = express();
    
    app.get('/calculate', async (req, res) => {
        const { numbers } = req.body;
        
        try {
            const result = await runWorker('./worker.js', { numbers });
            res.json({ result });
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    });
    
    app.listen(3000, () => {
        console.log('Server running on port 3000');
    });
    
    // Helper function to run worker
    function runWorker(workerScript, data) {
        return new Promise((resolve, reject) => {
            const worker = new Worker(workerScript, { workerData: data });
            
            worker.on('message', resolve);
            worker.on('error', reject);
            worker.on('exit', (code) => {
                if (code !== 0) {
                    reject(new Error(`Worker stopped with exit code ${code}`));
                }
            });
        });
    }
    
} else {
    // Worker thread logic would be in separate file
    console.log('This is a worker thread');
}
```

**worker.js (separate file):**
```javascript
const { parentPort, workerData } = require('worker_threads');

function heavyCalculation(numbers) {
    // CPU-intensive calculation
    let result = 0;
    for (let i = 0; i < numbers.length; i++) {
        for (let j = 0; j < 1000000; j++) {
            result += Math.sqrt(numbers[i] * j);
        }
    }
    return result;
}

// Perform calculation and send result back
const result = heavyCalculation(workerData.numbers);
parentPort.postMessage(result);
```

**Worker Thread Pool:**
```javascript
// worker-pool.js
const { Worker } = require('worker_threads');

class WorkerPool {
    constructor(size, workerScript) {
        this.size = size;
        this.workerScript = workerScript;
        this.workers = [];
        this.queue = [];
        this.initWorkers();
    }
    
    initWorkers() {
        for (let i = 0; i < this.size; i++) {
            const worker = new Worker(this.workerScript);
            worker.isAvailable = true;
            this.workers.push(worker);
        }
    }
    
    async execute(data) {
        return new Promise((resolve, reject) => {
            const availableWorker = this.workers.find(w => w.isAvailable);
            
            if (availableWorker) {
                this.runTask(availableWorker, data, resolve, reject);
            } else {
                this.queue.push({ data, resolve, reject });
            }
        });
    }
    
    runTask(worker, data, resolve, reject) {
        worker.isAvailable = false;
        
        worker.postMessage(data);
        
        const handleMessage = (result) => {
            worker.off('message', handleMessage);
            worker.off('error', handleError);
            worker.isAvailable = true;
            
            // Process queue
            if (this.queue.length > 0) {
                const { data: queuedData, resolve: queuedResolve, reject: queuedReject } = this.queue.shift();
                this.runTask(worker, queuedData, queuedResolve, queuedReject);
            }
            
            resolve(result);
        };
        
        const handleError = (error) => {
            worker.off('message', handleMessage);
            worker.off('error', handleError);
            worker.isAvailable = true;
            reject(error);
        };
        
        worker.on('message', handleMessage);
        worker.on('error', handleError);
    }
    
    destroy() {
        this.workers.forEach(worker => worker.terminate());
    }
}

module.exports = WorkerPool;
```

**Use Cases:**
- **Cluster**: Web servers, load distribution, fault tolerance
- **Worker Threads**: CPU-intensive calculations, data processing, image processing

---

### **3. Cách xử lý upload file trong Node.js? Có vấn đề gì với memory?**

**A: FILE UPLOAD HANDLING**

**1. Using Multer (Memory issues với large files):**
```javascript
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;

// ❌ BAD: Memory storage - loads entire file to memory
const memoryStorage = multer.memoryStorage();
const uploadMemory = multer({ 
    storage: memoryStorage,
    limits: { fileSize: 5 * 1024 * 1024 } // 5MB limit
});

app.post('/upload-memory', uploadMemory.single('file'), (req, res) => {
    // File in req.file.buffer - entire file in memory!
    console.log('File size:', req.file.buffer.length);
    res.json({ message: 'File uploaded to memory' });
});
```

**2. Disk Storage (Better for large files):**
```javascript
// ✅ GOOD: Disk storage - streams to disk
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        const uploadDir = './uploads';
        cb(null, uploadDir);
    },
    filename: (req, file, cb) => {
        const uniqueName = Date.now() + '-' + Math.round(Math.random() * 1E9);
        const extension = path.extname(file.originalname);
        cb(null, uniqueName + extension);
    }
});

const upload = multer({
    storage,
    limits: {
        fileSize: 100 * 1024 * 1024, // 100MB
        files: 10 // Maximum 10 files
    },
    fileFilter: (req, file, cb) => {
        // Validate file type
        const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
        if (allowedTypes.includes(file.mimetype)) {
            cb(null, true);
        } else {
            cb(new Error('Invalid file type'), false);
        }
    }
});

app.post('/upload-disk', upload.array('files', 10), async (req, res) => {
    try {
        const files = req.files.map(file => ({
            filename: file.filename,
            originalName: file.originalname,
            size: file.size,
            path: file.path
        }));
        
        res.json({ 
            message: 'Files uploaded successfully',
            files 
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

**3. Streaming Upload (Memory efficient):**
```javascript
const stream = require('stream');
const { pipeline } = require('stream/promises');

// Custom streaming handler
app.post('/upload-stream', (req, res) => {
    const uploadPath = `./uploads/${Date.now()}-upload.bin`;
    const writeStream = fs.createWriteStream(uploadPath);
    
    let uploadedBytes = 0;
    const maxSize = 100 * 1024 * 1024; // 100MB
    
    const transformStream = new stream.Transform({
        transform(chunk, encoding, callback) {
            uploadedBytes += chunk.length;
            
            if (uploadedBytes > maxSize) {
                return callback(new Error('File too large'));
            }
            
            // Process chunk if needed (virus scan, compression, etc.)
            callback(null, chunk);
        }
    });
    
    pipeline(req, transformStream, writeStream)
        .then(() => {
            res.json({
                message: 'File uploaded via streaming',
                size: uploadedBytes,
                path: uploadPath
            });
        })
        .catch((error) => {
            // Cleanup on error
            fs.unlink(uploadPath).catch(() => {});
            res.status(500).json({ error: error.message });
        });
});
```

**4. AWS S3 Direct Upload:**
```javascript
const AWS = require('aws-sdk');
const { v4: uuidv4 } = require('uuid');

const s3 = new AWS.S3({
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    region: process.env.AWS_REGION
});

// Generate signed URL for direct upload
app.post('/upload-url', (req, res) => {
    const { fileName, fileType } = req.body;
    const key = `uploads/${uuidv4()}-${fileName}`;
    
    const params = {
        Bucket: process.env.S3_BUCKET,
        Key: key,
        ContentType: fileType,
        Expires: 300, // 5 minutes
        ACL: 'private'
    };
    
    s3.getSignedUrl('putObject', params, (error, url) => {
        if (error) {
            return res.status(500).json({ error: error.message });
        }
        
        res.json({
            uploadUrl: url,
            key: key
        });
    });
});

// Upload to S3 via stream
app.post('/upload-s3-stream', upload.single('file'), async (req, res) => {
    try {
        const key = `uploads/${uuidv4()}-${req.file.originalname}`;
        
        const uploadParams = {
            Bucket: process.env.S3_BUCKET,
            Key: key,
            Body: fs.createReadStream(req.file.path),
            ContentType: req.file.mimetype
        };
        
        const result = await s3.upload(uploadParams).promise();
        
        // Delete temporary file
        await fs.unlink(req.file.path);
        
        res.json({
            message: 'File uploaded to S3',
            url: result.Location,
            key: result.Key
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

**5. Progress Tracking:**
```javascript
const busboy = require('busboy');

app.post('/upload-progress', (req, res) => {
    const bb = busboy({ headers: req.headers });
    const uploads = [];
    
    bb.on('file', (name, file, info) => {
        const { filename, encoding, mimeType } = info;
        const saveTo = `./uploads/${Date.now()}-${filename}`;
        const writeStream = fs.createWriteStream(saveTo);
        
        let uploadedBytes = 0;
        let totalBytes = 0;
        
        file.on('data', (data) => {
            uploadedBytes += data.length;
            
            // Send progress update (via WebSocket or SSE)
            const progress = Math.round((uploadedBytes / totalBytes) * 100);
            console.log(`Upload progress: ${progress}%`);
        });
        
        file.on('end', () => {
            uploads.push({
                filename,
                size: uploadedBytes,
                path: saveTo
            });
        });
        
        file.pipe(writeStream);
    });
    
    bb.on('finish', () => {
        res.json({
            message: 'Upload completed',
            files: uploads
        });
    });
    
    req.pipe(bb);
});
```

**MEMORY ISSUES & SOLUTIONS:**

**Vấn đề:**
- Memory storage loads entire file vào RAM
- Large files có thể gây out-of-memory
- Multiple concurrent uploads = memory spike

**Giải pháp:**
```javascript
// 1. Use disk storage instead of memory
const storage = multer.diskStorage({...});

// 2. Set reasonable limits
const upload = multer({
    storage,
    limits: {
        fileSize: 50 * 1024 * 1024, // 50MB max
        files: 5 // Max 5 files concurrent
    }
});

// 3. Stream processing for large files
const { pipeline } = require('stream/promises');

// 4. Cleanup temporary files
app.use((error, req, res, next) => {
    if (req.files) {
        req.files.forEach(file => {
            fs.unlink(file.path).catch(() => {});
        });
    }
    next(error);
});

// 5. Process uploads in background
const Queue = require('bull');
const uploadQueue = new Queue('file upload processing');

uploadQueue.process(async (job) => {
    const { filePath, userId } = job.data;
    
    // Process file (resize, scan, etc.)
    await processFile(filePath);
    
    // Move to final location
    await moveToStorage(filePath);
    
    // Update database
    await updateUserFile(userId, filePath);
});
```

---

### **4. Làm thế nào để bảo mật API bằng JWT? JWT so với session-based auth khác gì?**

**A: JWT SECURITY & COMPARISON**

**JWT IMPLEMENTATION:**
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const rateLimit = require('express-rate-limit');

// JWT Configuration
const JWT_CONFIG = {
    secret: process.env.JWT_SECRET, // Strong secret key
    accessTokenExpiry: '15m',
    refreshTokenExpiry: '7d',
    algorithm: 'HS256'
};

// Login endpoint
app.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Validate input
        if (!email || !password) {
            return res.status(400).json({ error: 'Email and password required' });
        }
        
        // Find user
        const user = await User.findOne({ email });
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Generate tokens
        const payload = {
            userId: user._id,
            email: user.email,
            role: user.role
        };
        
        const accessToken = jwt.sign(payload, JWT_CONFIG.secret, {
            expiresIn: JWT_CONFIG.accessTokenExpiry,
            algorithm: JWT_CONFIG.algorithm
        });
        
        const refreshToken = jwt.sign(
            { userId: user._id }, 
            JWT_CONFIG.secret, 
            { expiresIn: JWT_CONFIG.refreshTokenExpiry }
        );
        
        // Store refresh token in database
        await User.findByIdAndUpdate(user._id, { 
            refreshToken,
            lastLogin: new Date()
        });
        
        // Set secure cookies
        res.cookie('accessToken', accessToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 15 * 60 * 1000 // 15 minutes
        });
        
        res.cookie('refreshToken', refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
        });
        
        res.json({
            message: 'Login successful',
            user: {
                id: user._id,
                email: user.email,
                role: user.role
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Authentication middleware
function authenticateToken(req, res, next) {
    // Try to get token from cookie first, then header
    const token = req.cookies.accessToken || 
                  (req.headers.authorization && req.headers.authorization.split(' ')[1]);
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    jwt.verify(token, JWT_CONFIG.secret, (err, decoded) => {
        if (err) {
            if (err.name === 'TokenExpiredError') {
                return res.status(401).json({ error: 'Token expired' });
            }
            return res.status(403).json({ error: 'Invalid token' });
        }
        
        req.user = decoded;
        next();
    });
}

// Refresh token endpoint
app.post('/refresh-token', async (req, res) => {
    try {
        const refreshToken = req.cookies.refreshToken;
        
        if (!refreshToken) {
            return res.status(401).json({ error: 'Refresh token required' });
        }
        
        // Verify refresh token
        const decoded = jwt.verify(refreshToken, JWT_CONFIG.secret);
        
        // Check if token exists in database
        const user = await User.findById(decoded.userId);
        if (!user || user.refreshToken !== refreshToken) {
            return res.status(403).json({ error: 'Invalid refresh token' });
        }
        
        // Generate new access token
        const newAccessToken = jwt.sign({
            userId: user._id,
            email: user.email,
            role: user.role
        }, JWT_CONFIG.secret, {
            expiresIn: JWT_CONFIG.accessTokenExpiry
        });
        
        res.cookie('accessToken', newAccessToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 15 * 60 * 1000
        });
        
        res.json({ message: 'Token refreshed' });
        
    } catch (error) {
        res.status(403).json({ error: 'Token refresh failed' });
    }
});

// Logout endpoint
app.post('/logout', authenticateToken, async (req, res) => {
    try {
        // Remove refresh token from database
        await User.findByIdAndUpdate(req.user.userId, {
            refreshToken: null
        });
        
        // Clear cookies
        res.clearCookie('accessToken');
        res.clearCookie('refreshToken');
        
        res.json({ message: 'Logged out successfully' });
    } catch (error) {
        res.status(500).json({ error: 'Logout failed' });
    }
});
```

**JWT SECURITY BEST PRACTICES:**
```javascript
// 1. Strong secret key generation
const crypto = require('crypto');
const jwtSecret = crypto.randomBytes(64).toString('hex');

// 2. Token blacklisting for logout
const Redis = require('redis');
const redis = Redis.createClient();

async function blacklistToken(token) {
    const decoded = jwt.decode(token);
    const ttl = decoded.exp - Math.floor(Date.now() / 1000);
    if (ttl > 0) {
        await redis.setex(`blacklist:${token}`, ttl, 'true');
    }
}

// Enhanced auth middleware with blacklist check
async function authenticateTokenSecure(req, res, next) {
    const token = req.cookies.accessToken || 
                  (req.headers.authorization && req.headers.authorization.split(' ')[1]);
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    // Check if token is blacklisted
    const isBlacklisted = await redis.get(`blacklist:${token}`);
    if (isBlacklisted) {
        return res.status(401).json({ error: 'Token is blacklisted' });
    }
    
    jwt.verify(token, JWT_CONFIG.secret, (err, decoded) => {
        if (err) {
            return res.status(403).json({ error: 'Invalid token' });
        }
        req.user = decoded;
        next();
    });
}

// 3. Role-based access control
function authorizeRole(...roles) {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        next();
    };
}

// 4. Rate limiting for auth endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts per window
    message: 'Too many login attempts',
    standardHeaders: true,
    legacyHeaders: false,
    handler: (req, res) => {
        res.status(429).json({
            error: 'Too many login attempts, please try again later'
        });
    }
});

app.use('/login', authLimiter);
```

**JWT VS SESSION-BASED AUTH:**

| **JWT** | **Session-Based** |
|---------|------------------|
| **Stateless** | **Stateful** |
| Token stored client-side | Session ID stored client-side |
| User info in token | User info in server memory/database |
| No server storage needed | Requires server session storage |
| Scales horizontally easily | Harder to scale (sticky sessions) |
| Can't revoke immediately | Immediate revocation |
| Larger payload size | Small session ID |
| No server round-trip | Server lookup required |

**Session-Based Implementation:**
```javascript
const session = require('express-session');
const MongoStore = require('connect-mongo');

// Session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({
        mongoUrl: process.env.MONGODB_URL,
        touchAfter: 24 * 3600 // lazy session update
    }),
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 1000 * 60 * 60 * 24 // 24 hours
    }
}));

// Session-based login
app.post('/session-login', async (req, res) => {
    const { email, password } = req.body;
    
    const user = await authenticateUser(email, password);
    if (user) {
        req.session.userId = user._id;
        req.session.role = user.role;
        res.json({ message: 'Login successful' });
    } else {
        res.status(401).json({ error: 'Invalid credentials' });
    }
});

// Session-based auth middleware
function requireSession(req, res, next) {
    if (req.session && req.session.userId) {
        next();
    } else {
        res.status(401).json({ error: 'Authentication required' });
    }
}

// Session logout
app.post('/session-logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'Logout failed' });
        }
        res.clearCookie('connect.sid');
        res.json({ message: 'Logged out' });
    });
});
```

**When to use what:**
- **JWT**: Microservices, mobile apps, stateless architecture
- **Sessions**: Traditional web apps, simpler security requirements

**Security Considerations:**
- Store JWT in httpOnly cookies, not localStorage
- Use short expiration times for access tokens
- Implement proper refresh token rotation
- Add rate limiting to prevent brute force
- Validate and sanitize all inputs
- Use HTTPS in production
- Monitor for suspicious activities

---

### **5. Khi nào nên dùng Redis? Các use case phổ biến?**

**A: REDIS USE CASES & IMPLEMENTATION**

**REDIS LÀ GÌ:**
- In-memory data structure store
- Key-value database với performance cao
- Hỗ trợ nhiều data types: strings, hashes, lists, sets, sorted sets
- Persistence options: RDB snapshots, AOF logging

**KHI NÀO DÙNG REDIS:**

**1. Caching Layer:**
```javascript
const redis = require('redis');
const client = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD
});

// Cache middleware
async function cacheMiddleware(req, res, next) {
    const key = `cache:${req.originalUrl}`;
    
    try {
        const cachedData = await client.get(key);
        if (cachedData) {
            return res.json(JSON.parse(cachedData));
        }
        
        // Store original res.json
        const originalJson = res.json.bind(res);
        res.json = (data) => {
            // Cache for 5 minutes
            client.setex(key, 300, JSON.stringify(data));
            return originalJson(data);
        };
        
        next();
    } catch (error) {
        console.error('Cache error:', error);
        next(); // Continue without cache
    }
}

// Usage
app.get('/api/products', cacheMiddleware, async (req, res) => {
    const products = await Product.find().populate('category');
    res.json(products);
});

// Cache invalidation
app.post('/api/products', async (req, res) => {
    const product = await Product.create(req.body);
    
    // Invalidate related caches
    const keys = await client.keys('cache:/api/products*');
    if (keys.length > 0) {
        await client.del(keys);
    }
    
    res.json(product);
});
```

**2. Session Storage:**
```javascript
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
    store: new RedisStore({ client }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 1000 * 60 * 60 * 24 // 24 hours
    }
}));

// Custom session operations
async function setUserSession(userId, sessionData) {
    const key = `session:user:${userId}`;
    await client.hset(key, sessionData);
    await client.expire(key, 3600); // 1 hour
}

async function getUserSession(userId) {
    const key = `session:user:${userId}`;
    return await client.hgetall(key);
}
```

**3. Rate Limiting:**
```javascript
// Simple rate limiting
async function rateLimiter(req, res, next) {
    const key = `rate_limit:${req.ip}`;
    const limit = 100; // requests per window
    const window = 900; // 15 minutes
    
    try {
        const current = await client.get(key);
        
        if (!current) {
            await client.setex(key, window, 1);
            req.remainingRequests = limit - 1;
        } else {
            const requests = parseInt(current);
            if (requests >= limit) {
                return res.status(429).json({
                    error: 'Rate limit exceeded',
                    retryAfter: await client.ttl(key)
                });
            }
            
            await client.incr(key);
            req.remainingRequests = limit - requests - 1;
        }
        
        res.set('X-RateLimit-Limit', limit);
        res.set('X-RateLimit-Remaining', req.remainingRequests);
        next();
    } catch (error) {
        console.error('Rate limit error:', error);
        next(); // Continue without rate limiting
    }
}

// Sliding window rate limiter
class SlidingWindowRateLimiter {
    constructor(redis, maxRequests, windowMs) {
        this.redis = redis;
        this.maxRequests = maxRequests;
        this.windowMs = windowMs;
    }
    
    async isAllowed(identifier) {
        const key = `sliding:${identifier}`;
        const now = Date.now();
        const cutoff = now - this.windowMs;
        
        // Remove old entries and add current request
        const multi = this.redis.multi();
        multi.zremrangebyscore(key, '-inf', cutoff);
        multi.zadd(key, now, now);
        multi.zcard(key);
        multi.expire(key, Math.ceil(this.windowMs / 1000));
        
        const results = await multi.exec();
        const requestCount = results[2][1];
        
        return requestCount <= this.maxRequests;
    }
}
```

**4. Real-time Features (Pub/Sub):**
```javascript
// Publisher
async function notifyUsers(event, data) {
    await client.publish('notifications', JSON.stringify({
        event,
        data,
        timestamp: Date.now()
    }));
}

// Subscriber
const subscriber = redis.createClient();
subscriber.on('message', (channel, message) => {
    const notification = JSON.parse(message);
    
    // Broadcast to connected WebSocket clients
    io.emit(notification.event, notification.data);
});

subscriber.subscribe('notifications');

// Real-time chat
app.post('/api/chat/message', async (req, res) => {
    const { roomId, userId, message } = req.body;
    
    const chatMessage = {
        id: Date.now(),
        roomId,
        userId,
        message,
        timestamp: new Date()
    };
    
    // Store in Redis list
    await client.lpush(`chat:${roomId}`, JSON.stringify(chatMessage));
    await client.ltrim(`chat:${roomId}`, 0, 99); // Keep last 100 messages
    
    // Publish to subscribers
    await client.publish(`chat:${roomId}`, JSON.stringify(chatMessage));
    
    res.json(chatMessage);
});
```

**5. Job Queues:**
```javascript
const Queue = require('bull');

// Create queues with Redis
const emailQueue = new Queue('email processing', {
    redis: {
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT
    }
});

const imageQueue = new Queue('image processing', {
    redis: {
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT
    }
});

// Add jobs to queue
app.post('/api/send-email', async (req, res) => {
    const { to, subject, body } = req.body;
    
    const job = await emailQueue.add('send-email', {
        to,
        subject,
        body
    }, {
        attempts: 3,
        backoff: {
            type: 'exponential',
            delay: 2000
        }
    });
    
    res.json({ jobId: job.id });
});

// Process jobs
emailQueue.process('send-email', async (job) => {
    const { to, subject, body } = job.data;
    
    // Send email logic
    await sendEmail(to, subject, body);
    
    return { success: true, sentTo: to };
});

// Job status tracking
app.get('/api/job-status/:jobId', async (req, res) => {
    const job = await emailQueue.getJob(req.params.jobId);
    
    if (!job) {
        return res.status(404).json({ error: 'Job not found' });
    }
    
    res.json({
        id: job.id,
        status: await job.getState(),
        progress: job.progress(),
        data: job.data
    });
});
```

**6. Distributed Locks:**
```javascript
// Distributed lock implementation
class RedisLock {
    constructor(redis, key, ttl = 10000) {
        this.redis = redis;
        this.key = `lock:${key}`;
        this.ttl = ttl;
        this.value = null;
    }
    
    async acquire() {
        this.value = Date.now() + Math.random();
        
        const result = await this.redis.set(
            this.key, 
            this.value, 
            'PX', 
            this.ttl, 
            'NX'
        );
        
        return result === 'OK';
    }
    
    async release() {
        const script = `
            if redis.call("get", KEYS[1]) == ARGV[1] then
                return redis.call("del", KEYS[1])
            else
                return 0
            end
        `;
        
        const result = await this.redis.eval(script, 1, this.key, this.value);
        return result === 1;
    }
}

// Usage for preventing duplicate operations
app.post('/api/process-payment', async (req, res) => {
    const { orderId } = req.body;
    const lock = new RedisLock(client, `payment:${orderId}`, 30000);
    
    try {
        const acquired = await lock.acquire();
        if (!acquired) {
            return res.status(409).json({ error: 'Payment already being processed' });
        }
        
        // Process payment
        const result = await processPayment(orderId);
        
        res.json(result);
    } catch (error) {
        res.status(500).json({ error: error.message });
    } finally {
        await lock.release();
    }
});
```

**7. Analytics & Counters:**
```javascript
// Page view counters
async function trackPageView(page, userId = null) {
    const today = new Date().toISOString().split('T')[0];
    
    // Daily page views
    await client.hincrby(`page_views:${today}`, page, 1);
    
    // User tracking
    if (userId) {
        await client.sadd(`unique_visitors:${today}`, userId);
        await client.hincrby(`user_page_views:${userId}:${today}`, page, 1);
    }
    
    // Real-time stats
    await client.incr(`realtime_views:${page}`);
    await client.expire(`realtime_views:${page}`, 3600); // 1 hour
}

// Get analytics
app.get('/api/analytics/page-views', async (req, res) => {
    const { date = new Date().toISOString().split('T')[0] } = req.query;
    
    const [pageViews, uniqueVisitors] = await Promise.all([
        client.hgetall(`page_views:${date}`),
        client.scard(`unique_visitors:${date}`)
    ]);
    
    res.json({
        date,
        pageViews,
        uniqueVisitors
    });
});
```

**REDIS BEST PRACTICES:**

```javascript
// Connection management
const redis = require('redis');

class RedisManager {
    constructor() {
        this.client = null;
        this.subscriber = null;
        this.isConnected = false;
    }
    
    async connect() {
        try {
            this.client = redis.createClient({
                host: process.env.REDIS_HOST,
                port: process.env.REDIS_PORT,
                password: process.env.REDIS_PASSWORD,
                retry_strategy: (times) => {
                    const delay = Math.min(times * 50, 2000);
                    return delay;
                }
            });
            
            this.client.on('error', (err) => {
                console.error('Redis Client Error:', err);
                this.isConnected = false;
            });
            
            this.client.on('connect', () => {
                console.log('Connected to Redis');
                this.isConnected = true;
            });
            
            await this.client.connect();
        } catch (error) {
            console.error('Redis connection failed:', error);
            throw error;
        }
    }
    
    async gracefulShutdown() {
        if (this.client) {
            await this.client.quit();
        }
        if (this.subscriber) {
            await this.subscriber.quit();
        }
    }
}

// Memory optimization
async function optimizeMemory() {
    // Use appropriate data types
    await client.hset('user:1', 'name', 'John', 'age', 30); // Better than multiple keys
    
    // Set expiration for temporary data
    await client.setex('temp:data', 3600, 'value');
    
    // Use compression for large values
    const zlib = require('zlib');
    const compressed = zlib.gzipSync(JSON.stringify(largeData));
    await client.set('large:data', compressed);
    
    // Pipeline for multiple operations
    const pipeline = client.multi();
    pipeline.set('key1', 'value1');
    pipeline.set('key2', 'value2');
    pipeline.incr('counter');
    await pipeline.exec();
}
```

**USE CASES SUMMARY:**
- ✅ **Caching**: Database query results, API responses
- ✅ **Sessions**: User session storage
- ✅ **Rate Limiting**: API throttling, DDoS protection  
- ✅ **Real-time**: Chat, notifications, live updates
- ✅ **Queues**: Background job processing
- ✅ **Locks**: Distributed locking, preventing race conditions
- ✅ **Analytics**: Counters, metrics, leaderboards
- ✅ **Temporary Storage**: OTP codes, temporary tokens

---

### **6. Trình bày cách bạn xử lý rate limiting và chống brute force attack.**

**A: RATE LIMITING & BRUTE FORCE PROTECTION**

**1. EXPRESS-RATE-LIMIT (Cơ bản):**
```javascript
const rateLimit = require('express-rate-limit');

// Basic rate limiting
const generalLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: {
        error: 'Too many requests from this IP, please try again later.',
        retryAfter: '15 minutes'
    },
    standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
    legacyHeaders: false, // Disable the `X-RateLimit-*` headers
    handler: (req, res) => {
        res.status(429).json({
            error: 'Rate limit exceeded',
            retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
        });
    }
});

// Strict rate limiting for authentication
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts per window
    skipSuccessfulRequests: true, // Don't count successful requests
    skipFailedRequests: false, // Do count failed requests
    keyGenerator: (req) => {
        return req.ip + ':' + (req.body.email || req.body.username || '');
    },
    handler: (req, res) => {
        res.status(429).json({
            error: 'Too many authentication attempts',
            retryAfter: Math.ceil((req.rateLimit.resetTime - Date.now()) / 1000)
        });
    }
});

app.use('/api/', generalLimiter);
app.use('/auth/login', authLimiter);
app.use('/auth/register', authLimiter);
```

**2. REDIS-BASED RATE LIMITING (Advanced):**
```javascript
const Redis = require('redis');
const redis = Redis.createClient();

class AdvancedRateLimiter {
    constructor(redis) {
        this.redis = redis;
    }
    
    // Fixed window rate limiting
    async fixedWindow(key, limit, windowSeconds) {
        const current = await this.redis.get(key);
        
        if (!current) {
            await this.redis.setex(key, windowSeconds, 1);
            return { allowed: true, remaining: limit - 1, resetTime: Date.now() + windowSeconds * 1000 };
        }
        
        const count = parseInt(current);
        if (count >= limit) {
            const ttl = await this.redis.ttl(key);
            return { 
                allowed: false, 
                remaining: 0, 
                resetTime: Date.now() + ttl * 1000,
                retryAfter: ttl 
            };
        }
        
        await this.redis.incr(key);
        const ttl = await this.redis.ttl(key);
        return { 
            allowed: true, 
            remaining: limit - count - 1, 
            resetTime: Date.now() + ttl * 1000 
        };
    }
    
    // Sliding window rate limiting
    async slidingWindow(key, limit, windowMs) {
        const now = Date.now();
        const cutoff = now - windowMs;
        
        const multi = this.redis.multi();
        // Remove old entries
        multi.zremrangebyscore(key, '-inf', cutoff);
        // Add current request
        multi.zadd(key, now, now);
        // Get count
        multi.zcard(key);
        // Set expiration
        multi.expire(key, Math.ceil(windowMs / 1000));
        
        const results = await multi.exec();
        const count = results[2][1];
        
        return {
            allowed: count <= limit,
            remaining: Math.max(0, limit - count),
            resetTime: now + windowMs
        };
    }
    
    // Token bucket rate limiting
    async tokenBucket(key, capacity, refillRate, tokensRequested = 1) {
        const now = Date.now();
        const bucketKey = `bucket:${key}`;
        
        const bucket = await this.redis.hmget(bucketKey, 'tokens', 'lastRefill');
        let tokens = parseFloat(bucket[0]) || capacity;
        let lastRefill = parseInt(bucket[1]) || now;
        
        // Calculate tokens to add
        const timePassed = (now - lastRefill) / 1000;
        const tokensToAdd = timePassed * refillRate;
        tokens = Math.min(capacity, tokens + tokensToAdd);
        
        if (tokens >= tokensRequested) {
            tokens -= tokensRequested;
            
            await this.redis.hmset(bucketKey, {
                tokens: tokens.toString(),
                lastRefill: now.toString()
            });
            await this.redis.expire(bucketKey, 3600);
            
            return { allowed: true, remaining: Math.floor(tokens) };
        }
        
        return { 
            allowed: false, 
            remaining: Math.floor(tokens),
            retryAfter: Math.ceil((tokensRequested - tokens) / refillRate)
        };
    }
}

// Rate limiting middleware
const rateLimiter = new AdvancedRateLimiter(redis);

async function createRateLimitMiddleware(options) {
    const { type = 'sliding', limit, window, keyGenerator } = options;
    
    return async (req, res, next) => {
        try {
            const key = keyGenerator ? keyGenerator(req) : `rate_limit:${req.ip}`;
            
            let result;
            switch (type) {
                case 'fixed':
                    result = await rateLimiter.fixedWindow(key, limit, window / 1000);
                    break;
                case 'sliding':
                    result = await rateLimiter.slidingWindow(key, limit, window);
                    break;
                case 'token':
                    result = await rateLimiter.tokenBucket(key, limit, options.refillRate || 1);
                    break;
                default:
                    result = await rateLimiter.slidingWindow(key, limit, window);
            }
            
            // Set rate limit headers
            res.set('X-RateLimit-Limit', limit);
            res.set('X-RateLimit-Remaining', result.remaining);
            res.set('X-RateLimit-Reset', result.resetTime);
            
            if (!result.allowed) {
                if (result.retryAfter) {
                    res.set('Retry-After', result.retryAfter);
                }
                return res.status(429).json({
                    error: 'Rate limit exceeded',
                    retryAfter: result.retryAfter
                });
            }
            
            next();
        } catch (error) {
            console.error('Rate limiting error:', error);
            next(); // Continue without rate limiting on error
        }
    };
}
```

**3. BRUTE FORCE PROTECTION:**
```javascript
class BruteForceProtection {
    constructor(redis) {
        this.redis = redis;
        this.maxAttempts = 5;
        this.lockoutDuration = 15 * 60; // 15 minutes
        this.attemptWindow = 60 * 60; // 1 hour
    }
    
    async recordAttempt(identifier, success = false) {
        const key = `bf:${identifier}`;
        const now = Date.now();
        
        if (success) {
            // Reset attempts on successful login
            await this.redis.del(key);
            return { blocked: false };
        }
        
        // Add failed attempt
        await this.redis.zadd(key, now, now);
        await this.redis.expire(key, this.attemptWindow);
        
        // Remove old attempts outside window
        const cutoff = now - (this.attemptWindow * 1000);
        await this.redis.zremrangebyscore(key, '-inf', cutoff);
        
        // Count recent attempts
        const attempts = await this.redis.zcard(key);
        
        if (attempts >= this.maxAttempts) {
            // Set lockout
            const lockoutKey = `lockout:${identifier}`;
            await this.redis.setex(lockoutKey, this.lockoutDuration, attempts);
            
            return {
                blocked: true,
                attempts,
                unlockTime: now + (this.lockoutDuration * 1000)
            };
        }
        
        return {
            blocked: false,
            attempts,
            remaining: this.maxAttempts - attempts
        };
    }
    
    async isBlocked(identifier) {
        const lockoutKey = `lockout:${identifier}`;
        const lockout = await this.redis.get(lockoutKey);
        
        if (lockout) {
            const ttl = await this.redis.ttl(lockoutKey);
            return {
                blocked: true,
                unlockTime: Date.now() + (ttl * 1000),
                attempts: parseInt(lockout)
            };
        }
        
        return { blocked: false };
    }
    
    async getAttemptCount(identifier) {
        const key = `bf:${identifier}`;
        const now = Date.now();
        const cutoff = now - (this.attemptWindow * 1000);
        
        await this.redis.zremrangebyscore(key, '-inf', cutoff);
        return await this.redis.zcard(key);
    }
}

const bruteForceProtection = new BruteForceProtection(redis);

// Brute force middleware
async function bruteForceMiddleware(req, res, next) {
    const identifier = req.ip + ':' + (req.body.email || req.body.username || '');
    
    try {
        const status = await bruteForceProtection.isBlocked(identifier);
        
        if (status.blocked) {
            return res.status(429).json({
                error: 'Account temporarily locked due to too many failed attempts',
                unlockTime: status.unlockTime,
                attempts: status.attempts
            });
        }
        
        req.bruteForceIdentifier = identifier;
        next();
    } catch (error) {
        console.error('Brute force protection error:', error);
        next(); // Continue without protection on error
    }
}

// Login with brute force protection
app.post('/auth/login', bruteForceMiddleware, async (req, res) => {
    const { email, password } = req.body;
    
    try {
        const user = await User.findOne({ email });
        const isValid = user && await bcrypt.compare(password, user.password);
        
        // Record attempt
        const result = await bruteForceProtection.recordAttempt(
            req.bruteForceIdentifier, 
            isValid
        );
        
        if (!isValid) {
            return res.status(401).json({
                error: 'Invalid credentials',
                attemptsRemaining: result.remaining,
                willLockAfter: result.blocked ? 0 : result.remaining
            });
        }
        
        // Successful login
        const token = generateJWT(user);
        res.json({ token, user: { id: user._id, email: user.email } });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});
```

**4. CAPTCHA INTEGRATION:**
```javascript
const axios = require('axios');

async function verifyCaptcha(captchaResponse, userIP) {
    try {
        const response = await axios.post('https://www.google.com/recaptcha/api/siteverify', null, {
            params: {
                secret: process.env.RECAPTCHA_SECRET_KEY,
                response: captchaResponse,
                remoteip: userIP
            }
        });
        
        return response.data.success;
    } catch (error) {
        console.error('Captcha verification failed:', error);
        return false;
    }
}

// Enhanced brute force protection with CAPTCHA
async function enhancedBruteForceMiddleware(req, res, next) {
    const identifier = req.ip + ':' + (req.body.email || req.body.username || '');
    
    try {
        const attempts = await bruteForceProtection.getAttemptCount(identifier);
        
        // Require CAPTCHA after 3 failed attempts
        if (attempts >= 3) {
            const { captchaResponse } = req.body;
            
            if (!captchaResponse) {
                return res.status(400).json({
                    error: 'CAPTCHA required',
                    requiresCaptcha: true
                });
            }
            
            const captchaValid = await verifyCaptcha(captchaResponse, req.ip);
            if (!captchaValid) {
                return res.status(400).json({
                    error: 'Invalid CAPTCHA',
                    requiresCaptcha: true
                });
            }
        }
        
        const status = await bruteForceProtection.isBlocked(identifier);
        if (status.blocked) {
            return res.status(429).json({
                error: 'Account temporarily locked',
                unlockTime: status.unlockTime
            });
        }
        
        req.bruteForceIdentifier = identifier;
        next();
    } catch (error) {
        console.error('Enhanced brute force protection error:', error);
        next();
    }
}
```

**5. PROGRESSIVE DELAYS:**
```javascript
class ProgressiveDelayProtection {
    constructor(redis) {
        this.redis = redis;
        this.baseDelay = 1000; // 1 second
        this.maxDelay = 60000; // 60 seconds
    }
    
    async calculateDelay(identifier) {
        const key = `delay:${identifier}`;
        const attempts = await this.redis.get(key) || 0;
        
        // Exponential backoff: delay = baseDelay * (2 ^ attempts)
        const delay = Math.min(
            this.baseDelay * Math.pow(2, parseInt(attempts)),
            this.maxDelay
        );
        
        await this.redis.incr(key);
        await this.redis.expire(key, 3600); // Reset after 1 hour
        
        return delay;
    }
    
    async resetDelay(identifier) {
        const key = `delay:${identifier}`;
        await this.redis.del(key);
    }
}

const progressiveDelay = new ProgressiveDelayProtection(redis);

// Progressive delay middleware
async function progressiveDelayMiddleware(req, res, next) {
    const identifier = req.ip + ':login';
    
    try {
        const delay = await progressiveDelay.calculateDelay(identifier);
        
        if (delay > 1000) {
            // Add artificial delay
            await new Promise(resolve => setTimeout(resolve, delay));
        }
        
        req.progressiveDelayIdentifier = identifier;
        next();
    } catch (error) {
        console.error('Progressive delay error:', error);
        next();
    }
}
```

**6. COMPREHENSIVE SECURITY MIDDLEWARE:**
```javascript
// Complete security middleware stack
async function securityMiddleware(req, res, next) {
    const identifier = req.ip + ':' + (req.body.email || req.body.username || '');
    
    try {
        // 1. Check if IP is blocked
        const ipBlocked = await redis.get(`blocked_ip:${req.ip}`);
        if (ipBlocked) {
            return res.status(403).json({ error: 'IP address blocked' });
        }
        
        // 2. Rate limiting
        const rateLimit = await rateLimiter.slidingWindow(
            `rate:${req.ip}`, 
            10, // 10 requests
            60000 // per minute
        );
        
        if (!rateLimit.allowed) {
            return res.status(429).json({ error: 'Rate limit exceeded' });
        }
        
        // 3. Brute force protection
        const bruteForce = await bruteForceProtection.isBlocked(identifier);
        if (bruteForce.blocked) {
            return res.status(429).json({
                error: 'Too many failed attempts',
                unlockTime: bruteForce.unlockTime
            });
        }
        
        // 4. Suspicious activity detection
        const suspicious = await detectSuspiciousActivity(req);
        if (suspicious.score > 0.8) {
            // Log suspicious activity
            console.warn('Suspicious activity detected:', {
                ip: req.ip,
                userAgent: req.get('User-Agent'),
                score: suspicious.score,
                reasons: suspicious.reasons
            });
            
            // Require additional verification
            return res.status(400).json({
                error: 'Additional verification required',
                requiresVerification: true
            });
        }
        
        req.securityContext = {
            identifier,
            suspiciousScore: suspicious.score
        };
        
        next();
    } catch (error) {
        console.error('Security middleware error:', error);
        next();
    }
}

async function detectSuspiciousActivity(req) {
    let score = 0;
    const reasons = [];
    
    // Check for common attack patterns
    const userAgent = req.get('User-Agent') || '';
    if (userAgent.includes('bot') || userAgent.includes('crawler')) {
        score += 0.3;
        reasons.push('Bot user agent');
    }
    
    // Check for unusual request patterns
    const recentRequests = await redis.zcard(`requests:${req.ip}`);
    if (recentRequests > 100) {
        score += 0.4;
        reasons.push('High request frequency');
    }
    
    // Check for geographic anomalies (if you have user location data)
    // Check for time-based anomalies
    const hour = new Date().getHours();
    if (hour < 6 || hour > 22) {
        score += 0.1;
        reasons.push('Unusual time');
    }
    
    return { score, reasons };
}
```

**SUMMARY - BEST PRACTICES:**
- ✅ Multiple layers: Rate limiting + Brute force + Progressive delays
- ✅ Redis-based storage for distributed environments  
- ✅ CAPTCHA integration after multiple attempts
- ✅ Proper error handling và fallback mechanisms
- ✅ Logging và monitoring suspicious activities
- ✅ Configurable thresholds và timeouts
- ✅ IP blocking for persistent attackers
- ✅ Account lockout với reasonable unlock times

---

### **7. Hãy mô tả cách triển khai WebSocket với Node.js.**

**A: WEBSOCKET IMPLEMENTATION WITH NODE.JS**

**1. BASIC WEBSOCKET WITH SOCKET.IO:**
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const cors = require('cors');

const app = express();
const server = http.createServer(app);

// Socket.IO setup with CORS
const io = socketIo(server, {
    cors: {
        origin: ["http://localhost:3000", "https://yourdomain.com"],
        methods: ["GET", "POST"],
        credentials: true
    },
    pingTimeout: 60000,
    pingInterval: 25000
});

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

// Store connected users
const connectedUsers = new Map();
const rooms = new Map();

// Socket.IO connection handling
io.on('connection', (socket) => {
    console.log(`New client connected: ${socket.id}`);
    
    // Authentication
    socket.on('authenticate', async (data) => {
        try {
            const { token } = data;
            const user = await verifyToken(token); // Your JWT verification
            
            socket.userId = user.id;
            socket.userEmail = user.email;
            connectedUsers.set(user.id, {
                socketId: socket.id,
                user: user,
                joinedAt: new Date()
            });
            
            socket.emit('authenticated', { user });
            
            // Broadcast user online status
            socket.broadcast.emit('user-online', { userId: user.id });
            
        } catch (error) {
            socket.emit('authentication-error', { error: 'Invalid token' });
            socket.disconnect();
        }
    });
    
    // Join room
    socket.on('join-room', async (data) => {
        try {
            const { roomId } = data;
            
            if (!socket.userId) {
                socket.emit('error', { message: 'Not authenticated' });
                return;
            }
            
            // Leave previous rooms
            const previousRooms = Array.from(socket.rooms).filter(room => room !== socket.id);
            previousRooms.forEach(room => {
                socket.leave(room);
                socket.to(room).emit('user-left', {
                    userId: socket.userId,
                    email: socket.userEmail
                });
            });
            
            // Join new room
            await socket.join(roomId);
            
            // Track room members
            if (!rooms.has(roomId)) {
                rooms.set(roomId, new Set());
            }
            rooms.get(roomId).add(socket.userId);
            
            // Notify room members
            socket.to(roomId).emit('user-joined', {
                userId: socket.userId,
                email: socket.userEmail
            });
            
            // Send room info
            const roomMembers = Array.from(rooms.get(roomId));
            socket.emit('room-joined', {
                roomId,
                members: roomMembers.length
            });
            
        } catch (error) {
            socket.emit('error', { message: 'Failed to join room' });
        }
    });
    
    // Handle messages
    socket.on('send-message', async (data) => {
        try {
            const { roomId, message, messageType = 'text' } = data;
            
            if (!socket.userId) {
                socket.emit('error', { message: 'Not authenticated' });
                return;
            }
            
            // Validate message
            if (!message || message.trim().length === 0) {
                socket.emit('error', { message: 'Message cannot be empty' });
                return;
            }
            
            // Create message object
            const messageObj = {
                id: Date.now() + Math.random(),
                userId: socket.userId,
                userEmail: socket.userEmail,
                message: message.trim(),
                messageType,
                timestamp: new Date(),
                roomId
            };
            
            // Save to database
            await saveMessage(messageObj);
            
            // Broadcast to room members
            io.to(roomId).emit('new-message', messageObj);
            
        } catch (error) {
            console.error('Message error:', error);
            socket.emit('error', { message: 'Failed to send message' });
        }
    });
    
    // Handle typing indicators
    socket.on('typing-start', (data) => {
        if (socket.userId) {
            socket.to(data.roomId).emit('user-typing', {
                userId: socket.userId,
                email: socket.userEmail
            });
        }
    });
    
    socket.on('typing-stop', (data) => {
        if (socket.userId) {
            socket.to(data.roomId).emit('user-stopped-typing', {
                userId: socket.userId,
                email: socket.userEmail
            });
        }
    });
    
    // Handle file sharing
    socket.on('file-share', async (data) => {
        try {
            const { roomId, fileName, fileUrl, fileSize, fileType } = data;
            
            if (!socket.userId) {
                socket.emit('error', { message: 'Not authenticated' });
                return;
            }
            
            const fileMessage = {
                id: Date.now() + Math.random(),
                userId: socket.userId,
                userEmail: socket.userEmail,
                messageType: 'file',
                fileName,
                fileUrl,
                fileSize,
                fileType,
                timestamp: new Date(),
                roomId
            };
            
            await saveMessage(fileMessage);
            io.to(roomId).emit('new-message', fileMessage);
            
        } catch (error) {
            socket.emit('error', { message: 'Failed to share file' });
        }
    });
    
    // Handle disconnect
    socket.on('disconnect', (reason) => {
        console.log(`Client disconnected: ${socket.id}, reason: ${reason}`);
        
        if (socket.userId) {
            // Remove from connected users
            connectedUsers.delete(socket.userId);
            
            // Remove from rooms
            const userRooms = Array.from(socket.rooms);
            userRooms.forEach(roomId => {
                if (rooms.has(roomId)) {
                    rooms.get(roomId).delete(socket.userId);
                    if (rooms.get(roomId).size === 0) {
                        rooms.delete(roomId);
                    } else {
                        socket.to(roomId).emit('user-left', {
                            userId: socket.userId,
                            email: socket.userEmail
                        });
                    }
                }
            });
            
            // Broadcast user offline status
            socket.broadcast.emit('user-offline', { userId: socket.userId });
        }
    });
    
    // Heartbeat to keep connection alive
    socket.on('ping', () => {
        socket.emit('pong');
    });
});

// Helper functions
async function verifyToken(token) {
    const jwt = require('jsonwebtoken');
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Get user from database
    const user = await User.findById(decoded.userId);
    if (!user) {
        throw new Error('User not found');
    }
    
    return user;
}

async function saveMessage(message) {
    // Save to MongoDB or your preferred database
    const Message = require('./models/Message');
    return await Message.create(message);
}

// REST API endpoints
app.get('/api/messages/:roomId', async (req, res) => {
    try {
        const { roomId } = req.params;
        const { page = 1, limit = 50 } = req.query;
        
        const messages = await Message.find({ roomId })
            .sort({ timestamp: -1 })
            .limit(limit * 1)
            .skip((page - 1) * limit);
            
        res.json(messages.reverse());
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch messages' });
    }
});

app.get('/api/online-users', (req, res) => {
    const users = Array.from(connectedUsers.values()).map(conn => ({
        userId: conn.user.id,
        email: conn.user.email,
        joinedAt: conn.joinedAt
    }));
    
    res.json(users);
});

server.listen(3001, () => {
    console.log('Server running on port 3001');
});
```

**2. NATIVE WEBSOCKET IMPLEMENTATION:**
```javascript
const WebSocket = require('ws');
const url = require('url');
const jwt = require('jsonwebtoken');

// Create WebSocket server
const wss = new WebSocket.Server({
    port: 8080,
    verifyClient: async (info) => {
        try {
            const query = url.parse(info.req.url, true).query;
            const token = query.token;
            
            if (!token) {
                return false;
            }
            
            // Verify JWT token
            jwt.verify(token, process.env.JWT_SECRET);
            return true;
        } catch (error) {
            return false;
        }
    }
});

// Store connections
const connections = new Map();
const rooms = new Map();

wss.on('connection', async (ws, req) => {
    const query = url.parse(req.url, true).query;
    const token = query.token;
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        const user = await User.findById(decoded.userId);
        
        ws.userId = user._id.toString();
        ws.userEmail = user.email;
        ws.isAlive = true;
        
        connections.set(ws.userId, ws);
        
        console.log(`User ${user.email} connected`);
        
        // Send welcome message
        ws.send(JSON.stringify({
            type: 'welcome',
            data: { user: { id: user._id, email: user.email } }
        }));
        
    } catch (error) {
        ws.close(1008, 'Authentication failed');
        return;
    }
    
    // Handle messages
    ws.on('message', async (data) => {
        try {
            const message = JSON.parse(data);
            
            switch (message.type) {
                case 'join-room':
                    await handleJoinRoom(ws, message.data);
                    break;
                    
                case 'leave-room':
                    await handleLeaveRoom(ws, message.data);
                    break;
                    
                case 'chat-message':
                    await handleChatMessage(ws, message.data);
                    break;
                    
                case 'typing':
                    handleTyping(ws, message.data);
                    break;
                    
                case 'ping':
                    ws.send(JSON.stringify({ type: 'pong' }));
                    break;
                    
                default:
                    ws.send(JSON.stringify({
                        type: 'error',
                        data: { message: 'Unknown message type' }
                    }));
            }
        } catch (error) {
            console.error('Message handling error:', error);
            ws.send(JSON.stringify({
                type: 'error',
                data: { message: 'Invalid message format' }
            }));
        }
    });
    
    // Handle connection close
    ws.on('close', () => {
        console.log(`User ${ws.userEmail} disconnected`);
        
        connections.delete(ws.userId);
        
        // Remove from all rooms
        for (const [roomId, members] of rooms.entries()) {
            if (members.has(ws.userId)) {
                members.delete(ws.userId);
                
                // Notify other room members
                broadcastToRoom(roomId, {
                    type: 'user-left',
                    data: { userId: ws.userId, email: ws.userEmail }
                }, ws.userId);
                
                if (members.size === 0) {
                    rooms.delete(roomId);
                }
            }
        }
    });
    
    // Heartbeat
    ws.on('pong', () => {
        ws.isAlive = true;
    });
});

// Helper functions
async function handleJoinRoom(ws, data) {
    const { roomId } = data;
    
    if (!rooms.has(roomId)) {
        rooms.set(roomId, new Set());
    }
    
    rooms.get(roomId).add(ws.userId);
    ws.currentRoom = roomId;
    
    // Notify room members
    broadcastToRoom(roomId, {
        type: 'user-joined',
        data: { userId: ws.userId, email: ws.userEmail }
    }, ws.userId);
    
    // Send room info to user
    ws.send(JSON.stringify({
        type: 'room-joined',
        data: {
            roomId,
            memberCount: rooms.get(roomId).size
        }
    }));
}

async function handleLeaveRoom(ws, data) {
    const { roomId } = data;
    
    if (rooms.has(roomId)) {
        rooms.get(roomId).delete(ws.userId);
        
        broadcastToRoom(roomId, {
            type: 'user-left',
            data: { userId: ws.userId, email: ws.userEmail }
        }, ws.userId);
        
        if (rooms.get(roomId).size === 0) {
            rooms.delete(roomId);
        }
    }
    
    ws.currentRoom = null;
}

async function handleChatMessage(ws, data) {
    const { roomId, message } = data;
    
    if (!ws.currentRoom || ws.currentRoom !== roomId) {
        ws.send(JSON.stringify({
            type: 'error',
            data: { message: 'Not in room' }
        }));
        return;
    }
    
    const messageObj = {
        id: Date.now() + Math.random(),
        userId: ws.userId,
        userEmail: ws.userEmail,
        message,
        timestamp: new Date(),
        roomId
    };
    
    // Save to database
    await saveMessage(messageObj);
    
    // Broadcast to room members
    broadcastToRoom(roomId, {
        type: 'new-message',
        data: messageObj
    });
}

function handleTyping(ws, data) {
    const { roomId, isTyping } = data;
    
    if (ws.currentRoom === roomId) {
        broadcastToRoom(roomId, {
            type: isTyping ? 'user-typing' : 'user-stopped-typing',
            data: { userId: ws.userId, email: ws.userEmail }
        }, ws.userId);
    }
}

function broadcastToRoom(roomId, message, excludeUserId = null) {
    if (!rooms.has(roomId)) return;
    
    const members = rooms.get(roomId);
    members.forEach(userId => {
        if (userId !== excludeUserId) {
            const connection = connections.get(userId);
            if (connection && connection.readyState === WebSocket.OPEN) {
                connection.send(JSON.stringify(message));
            }
        }
    });
}

// Heartbeat interval to detect dead connections
const heartbeatInterval = setInterval(() => {
    wss.clients.forEach((ws) => {
        if (!ws.isAlive) {
            console.log('Terminating dead connection');
            return ws.terminate();
        }
        
        ws.isAlive = false;
        ws.ping();
    });
}, 30000); // Every 30 seconds

wss.on('close', () => {
    clearInterval(heartbeatInterval);
});
```

**3. SCALED WEBSOCKET WITH REDIS ADAPTER:**
```javascript
const redis = require('redis');
const { createAdapter } = require('@socket.io/redis-adapter');

// Redis setup for scaling Socket.IO
const pubClient = redis.createClient({
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
});

const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));

// Custom room management with Redis
class RedisRoomManager {
    constructor(redis) {
        this.redis = redis;
    }
    
    async joinRoom(userId, roomId) {
        const userKey = `user_room:${userId}`;
        const roomKey = `room_members:${roomId}`;
        
        // Remove user from previous room
        const previousRoom = await this.redis.get(userKey);
        if (previousRoom) {
            await this.redis.srem(`room_members:${previousRoom}`, userId);
        }
        
        // Add to new room
        await this.redis.set(userKey, roomId);
        await this.redis.sadd(roomKey, userId);
        
        return await this.redis.scard(roomKey);
    }
    
    async leaveRoom(userId, roomId) {
        await this.redis.del(`user_room:${userId}`);
        await this.redis.srem(`room_members:${roomId}`, userId);
        
        return await this.redis.scard(`room_members:${roomId}`);
    }
    
    async getRoomMembers(roomId) {
        return await this.redis.smembers(`room_members:${roomId}`);
    }
    
    async getUserRoom(userId) {
        return await this.redis.get(`user_room:${userId}`);
    }
}

const roomManager = new RedisRoomManager(pubClient);

// Enhanced Socket.IO with Redis
io.on('connection', (socket) => {
    socket.on('join-room', async (data) => {
        const { roomId } = data;
        
        await socket.join(roomId);
        const memberCount = await roomManager.joinRoom(socket.userId, roomId);
        
        socket.emit('room-joined', { roomId, memberCount });
        socket.to(roomId).emit('user-joined', {
            userId: socket.userId,
            email: socket.userEmail
        });
    });
    
    // Persistent message storage
    socket.on('send-message', async (data) => {
        const { roomId, message } = data;
        
        const messageObj = {
            id: Date.now() + Math.random(),
            userId: socket.userId,
            userEmail: socket.userEmail,
            message,
            timestamp: new Date(),
            roomId
        };
        
        // Save to database
        await saveMessage(messageObj);
        
        // Save to Redis for quick access
        await pubClient.lpush(
            `room_messages:${roomId}`,
            JSON.stringify(messageObj)
        );
        
        // Keep only last 100 messages in Redis
        await pubClient.ltrim(`room_messages:${roomId}`, 0, 99);
        
        // Broadcast
        io.to(roomId).emit('new-message', messageObj);
    });
});
```

**4. CLIENT-SIDE IMPLEMENTATION:**
```javascript
// Frontend JavaScript
class WebSocketClient {
    constructor(url, token) {
        this.url = url;
        this.token = token;
        this.socket = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
        this.reconnectDelay = 1000;
        this.listeners = new Map();
    }
    
    connect() {
        return new Promise((resolve, reject) => {
            this.socket = io(this.url, {
                auth: { token: this.token },
                transports: ['websocket', 'polling']
            });
            
            this.socket.on('connect', () => {
                console.log('Connected to server');
                this.reconnectAttempts = 0;
                resolve();
            });
            
            this.socket.on('connect_error', (error) => {
                console.error('Connection error:', error);
                reject(error);
            });
            
            this.socket.on('disconnect', (reason) => {
                console.log('Disconnected:', reason);
                
                if (reason === 'io server disconnect') {
                    // Server disconnected, try to reconnect
                    this.reconnect();
                }
            });
            
            // Setup event listeners
            this.setupEventListeners();
        });
    }
    
    setupEventListeners() {
        // Custom event handling
        this.listeners.forEach((handler, event) => {
            this.socket.on(event, handler);
        });
    }
    
    on(event, handler) {
        this.listeners.set(event, handler);
        if (this.socket) {
            this.socket.on(event, handler);
        }
    }
    
    emit(event, data) {
        if (this.socket && this.socket.connected) {
            this.socket.emit(event, data);
        } else {
            console.warn('Socket not connected');
        }
    }
    
    async reconnect() {
        if (this.reconnectAttempts >= this.maxReconnectAttempts) {
            console.error('Max reconnection attempts reached');
            return;
        }
        
        this.reconnectAttempts++;
        const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1);
        
        console.log(`Attempting to reconnect (${this.reconnectAttempts}/${this.maxReconnectAttempts}) in ${delay}ms`);
        
        setTimeout(() => {
            this.connect().catch(() => this.reconnect());
        }, delay);
    }
    
    disconnect() {
        if (this.socket) {
            this.socket.disconnect();
        }
    }
}

// Usage
const client = new WebSocketClient('http://localhost:3001', userToken);

client.on('new-message', (message) => {
    displayMessage(message);
});

client.on('user-joined', (data) => {
    showNotification(`${data.email} joined the room`);
});

client.connect().then(() => {
    // Join a room
    client.emit('join-room', { roomId: 'general' });
    
    // Send a message
    document.getElementById('send-btn').onclick = () => {
        const message = document.getElementById('message-input').value;
        client.emit('send-message', {
            roomId: 'general',
            message
        });
    };
});
```

**WEBSOCKET BEST PRACTICES:**
- ✅ Authentication và authorization
- ✅ Rate limiting để prevent spam
- ✅ Heartbeat/ping-pong để detect dead connections
- ✅ Proper error handling và reconnection logic
- ✅ Room management với Redis for scaling
- ✅ Message persistence trong database
- ✅ CORS configuration
- ✅ Connection cleanup on disconnect

---

### **8. Làm sao để xử lý CORS trong Express.js?**

**A: CORS HANDLING IN EXPRESS.JS**

**1. CƠ BẢN VỀ CORS:**
CORS (Cross-Origin Resource Sharing) là security mechanism cho phép/chặn web browsers truy cập resources từ different domains, protocols, hoặc ports.

**Same-Origin Policy Examples:**
- ✅ `https://example.com` → `https://example.com/api` (Same origin)
- ❌ `https://example.com` → `http://example.com` (Different protocol)  
- ❌ `https://example.com` → `https://api.example.com` (Different subdomain)
- ❌ `https://example.com:3000` → `https://example.com:8080` (Different port)

**2. CORS MIDDLEWARE SETUP:**
```javascript
const express = require('express');
const cors = require('cors');

const app = express();

// Basic CORS - allows all origins (NOT recommended for production)
app.use(cors());

// OR manually set headers
app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
    
    if (req.method === 'OPTIONS') {
        res.sendStatus(200);
    } else {
        next();
    }
});
```

**3. PRODUCTION CORS CONFIGURATION:**
```javascript
// Secure CORS configuration for production
const corsOptions = {
    origin: function (origin, callback) {
        // Allowed origins
        const allowedOrigins = [
            'https://yourdomain.com',
            'https://www.yourdomain.com',
            'https://app.yourdomain.com',
            'http://localhost:3000', // Development
            'http://localhost:3001'
        ];
        
        // Allow requests with no origin (mobile apps, Postman, etc.)
        if (!origin) return callback(null, true);
        
        if (allowedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    
    // Allowed HTTP methods
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
    
    // Allowed headers
    allowedHeaders: [
        'Origin',
        'X-Requested-With',
        'Content-Type',
        'Accept',
        'Authorization',
        'Cache-Control',
        'X-API-Key'
    ],
    
    // Headers exposed to the client
    exposedHeaders: [
        'X-Total-Count',
        'X-Rate-Limit-Remaining',
        'X-Rate-Limit-Reset'
    ],
    
    // Allow credentials (cookies, authorization headers)
    credentials: true,
    
    // Preflight cache duration (in seconds)
    maxAge: 86400, // 24 hours
    
    // Handle preflight OPTIONS requests
    preflightContinue: false,
    optionsSuccessStatus: 204
};

app.use(cors(corsOptions));
```

**4. DYNAMIC CORS CONFIGURATION:**
```javascript
// Dynamic CORS based on environment
const getDynamicCorsOptions = () => {
    const baseOptions = {
        methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
        allowedHeaders: [
            'Origin', 'X-Requested-With', 'Content-Type', 
            'Accept', 'Authorization', 'X-API-Key'
        ],
        credentials: true,
        maxAge: 86400
    };
    
    if (process.env.NODE_ENV === 'production') {
        return {
            ...baseOptions,
            origin: [
                'https://yourdomain.com',
                'https://www.yourdomain.com',
                'https://app.yourdomain.com'
            ]
        };
    } else if (process.env.NODE_ENV === 'development') {
        return {
            ...baseOptions,
            origin: [
                'http://localhost:3000',
                'http://localhost:3001',
                'http://127.0.0.1:3000'
            ]
        };
    } else {
        // Testing environment - allow all
        return {
            ...baseOptions,
            origin: true
        };
    }
};

app.use(cors(getDynamicCorsOptions()));
```

**5. ROUTE-SPECIFIC CORS:**
```javascript
// Different CORS rules for different routes
const publicCorsOptions = {
    origin: true, // Allow all origins
    methods: ['GET'],
    credentials: false
};

const privateCorsOptions = {
    origin: [
        'https://yourdomain.com',
        'http://localhost:3000'
    ],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    credentials: true,
    allowedHeaders: ['Content-Type', 'Authorization']
};

const adminCorsOptions = {
    origin: 'https://admin.yourdomain.com',
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    credentials: true,
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Admin-Key']
};

// Apply different CORS to different routes
app.use('/api/public', cors(publicCorsOptions));
app.use('/api/private', cors(privateCorsOptions));
app.use('/api/admin', cors(adminCorsOptions));

// Specific endpoints
app.get('/api/health', cors({ origin: true }), (req, res) => {
    res.json({ status: 'OK' });
});

app.post('/api/upload', cors(privateCorsOptions), upload.single('file'), (req, res) => {
    // File upload logic
});
```

**6. ADVANCED CORS WITH CUSTOM LOGIC:**
```javascript
// Advanced CORS with custom validation
const advancedCorsOptions = {
    origin: function (origin, callback) {
        // Log all CORS requests for monitoring
        console.log(`CORS request from origin: ${origin}`);
        
        // Check if origin is in whitelist
        const whitelist = process.env.ALLOWED_ORIGINS?.split(',') || [];
        
        // Allow requests with no origin (mobile apps, etc.)
        if (!origin) {
            return callback(null, true);
        }
        
        // Dynamic origin checking
        if (isValidOrigin(origin)) {
            callback(null, true);
        } else {
            // Log blocked requests
            console.warn(`CORS blocked request from: ${origin}`);
            callback(new Error(`Origin ${origin} not allowed by CORS`));
        }
    },
    
    credentials: function (req, callback) {
        // Dynamic credentials based on request
        const needsCredentials = req.path.startsWith('/api/auth') || 
                                req.path.startsWith('/api/user');
        callback(null, needsCredentials);
    },
    
    methods: function (req, callback) {
        // Dynamic methods based on request
        let allowedMethods = ['GET', 'POST', 'OPTIONS'];
        
        if (req.headers.authorization) {
            allowedMethods.push('PUT', 'DELETE', 'PATCH');
        }
        
        callback(null, allowedMethods);
    }
};

function isValidOrigin(origin) {
    // Custom validation logic
    try {
        const url = new URL(origin);
        
        // Allow localhost for development
        if (url.hostname === 'localhost' || url.hostname === '127.0.0.1') {
            return process.env.NODE_ENV !== 'production';
        }
        
        // Check against domain patterns
        const allowedDomains = [
            'yourdomain.com',
            '*.yourdomain.com',
            'partner-domain.com'
        ];
        
        return allowedDomains.some(domain => {
            if (domain.startsWith('*.')) {
                const baseDomain = domain.slice(2);
                return url.hostname.endsWith(baseDomain);
            }
            return url.hostname === domain;
        });
        
    } catch (error) {
        return false;
    }
}

app.use(cors(advancedCorsOptions));
```

**7. HANDLING PREFLIGHT REQUESTS:**
```javascript
// Custom preflight handling
app.options('*', (req, res) => {
    const origin = req.headers.origin;
    const method = req.headers['access-control-request-method'];
    const headers = req.headers['access-control-request-headers'];
    
    // Validate preflight request
    if (isValidOrigin(origin) && isValidMethod(method)) {
        res.header('Access-Control-Allow-Origin', origin);
        res.header('Access-Control-Allow-Methods', method);
        res.header('Access-Control-Allow-Headers', headers);
        res.header('Access-Control-Allow-Credentials', 'true');
        res.header('Access-Control-Max-Age', '86400'); // 24 hours
        res.sendStatus(204);
    } else {
        res.sendStatus(403);
    }
});

function isValidMethod(method) {
    const allowedMethods = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'];
    return allowedMethods.includes(method);
}

// Global preflight handler
app.use((req, res, next) => {
    if (req.method === 'OPTIONS') {
        // Log preflight requests
        console.log(`Preflight request: ${req.headers.origin} -> ${req.path}`);
        
        // Custom preflight logic
        const origin = req.headers.origin;
        if (origin && isValidOrigin(origin)) {
            res.header('Access-Control-Allow-Origin', origin);
            res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
            res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
            res.header('Access-Control-Allow-Credentials', 'true');
            res.header('Access-Control-Max-Age', '86400');
            return res.sendStatus(204);
        }
    }
    next();
});
```

**8. CORS ERROR HANDLING:**
```javascript
// Global CORS error handler
app.use((error, req, res, next) => {
    if (error.message.includes('CORS')) {
        console.warn('CORS Error:', {
            origin: req.headers.origin,
            method: req.method,
            path: req.path,
            userAgent: req.headers['user-agent'],
            ip: req.ip
        });
        
        return res.status(403).json({
            error: 'CORS Error',
            message: 'Origin not allowed',
            code: 'CORS_NOT_ALLOWED'
        });
    }
    next(error);
});

// CORS monitoring middleware
const corsMonitoring = (req, res, next) => {
    const origin = req.headers.origin;
    
    if (origin && !isValidOrigin(origin)) {
        // Log suspicious CORS attempts
        console.warn('Blocked CORS request:', {
            origin,
            method: req.method,
            path: req.path,
            timestamp: new Date().toISOString(),
            ip: req.ip,
            userAgent: req.headers['user-agent']
        });
        
        // Optionally store in database for analysis
        // await logCorsAttempt(origin, req);
    }
    
    next();
};

app.use(corsMonitoring);
```

**9. TESTING CORS CONFIGURATION:**
```javascript
// Test endpoint to check CORS configuration
app.get('/api/cors-test', (req, res) => {
    res.json({
        message: 'CORS test successful',
        origin: req.headers.origin,
        method: req.method,
        headers: req.headers,
        timestamp: new Date().toISOString()
    });
});

// Debug CORS middleware
const debugCors = (req, res, next) => {
    if (process.env.NODE_ENV === 'development') {
        console.log('CORS Debug:', {
            origin: req.headers.origin,
            method: req.method,
            path: req.path,
            corsHeaders: {
                'access-control-request-method': req.headers['access-control-request-method'],
                'access-control-request-headers': req.headers['access-control-request-headers']
            }
        });
    }
    next();
};

app.use(debugCors);
```

**10. CORS WITH AUTHENTICATION:**
```javascript
// CORS configuration for authenticated requests
const authenticatedCorsOptions = {
    origin: function (origin, callback) {
        // More restrictive origins for authenticated endpoints
        const trustedOrigins = [
            'https://app.yourdomain.com',
            'https://dashboard.yourdomain.com'
        ];
        
        if (!origin || trustedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS for authenticated requests'));
        }
    },
    credentials: true, // Required for cookies/auth headers
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token']
};

// Apply to authenticated routes
app.use('/api/auth/*', cors(authenticatedCorsOptions));
app.use('/api/user/*', cors(authenticatedCorsOptions));

// Handle authentication with CORS
app.post('/api/login', cors(authenticatedCorsOptions), async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // Validate credentials
        const user = await authenticateUser(email, password);
        const token = generateJWT(user);
        
        // Set secure cookie
        res.cookie('authToken', token, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 24 * 60 * 60 * 1000 // 24 hours
        });
        
        res.json({ user: { id: user.id, email: user.email } });
    } catch (error) {
        res.status(401).json({ error: 'Invalid credentials' });
    }
});
```

**CORS BEST PRACTICES:**
- ✅ Never use `origin: "*"` with `credentials: true` trong production
- ✅ Specify exact origins thay vì wildcard
- ✅ Use environment-specific configurations
- ✅ Log và monitor CORS violations
- ✅ Set reasonable `maxAge` for preflight caching
- ✅ Validate origins dynamically when needed
- ✅ Handle preflight requests properly
- ✅ Use HTTPS trong production
- ✅ Implement proper error handling
- ✅ Test CORS configuration thoroughly
