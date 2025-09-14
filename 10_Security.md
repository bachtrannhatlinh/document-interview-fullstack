# 10. Security
- **Các lỗ hổng phổ biến:**
  - **XSS (Cross-Site Scripting):** Chèn mã độc vào web, thường gặp khi render dữ liệu không kiểm soát lên client. Phòng tránh: escape output, không render trực tiếp dữ liệu user nhập.
  - **CSRF (Cross-Site Request Forgery):** Kẻ tấn công lợi dụng session của user để gửi request giả mạo. Phòng tránh: dùng CSRF token, kiểm tra Origin/Referer.
  - **SQL Injection:** Chèn mã SQL vào query, thường gặp khi dùng query string trực tiếp với DB. Phòng tránh: dùng ORM, prepared statement, không nối chuỗi query.
  - **NoSQL Injection:** Tương tự SQLi nhưng với NoSQL (MongoDB, ...), ví dụ truyền object `{ "$gt": "" }` để bypass auth. Phòng tránh: validate input, không truyền object trực tiếp vào query.

- **Cách phòng tránh:**
  - **helmet:** Middleware bảo vệ HTTP headers cho Express/NestJS.
    ```js
    const helmet = require('helmet');
    app.use(helmet());
    ```
  - **sanitize:** Làm sạch input, loại bỏ ký tự nguy hiểm (dùng thư viện như express-validator, validator, mongo-sanitize).
    ```js
    const { sanitize } = require('express-validator');
    app.post('/user', sanitize('name').escape(), ...);
    ```
  - **validate input:** Kiểm tra dữ liệu đầu vào (type, length, format, ...), reject nếu không hợp lệ.
    ```js
    const { body, validationResult } = require('express-validator');
    app.post('/user', body('email').isEmail(), (req, res) => {
      const errors = validationResult(req);
      if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
      // ...
    });
    ```
  - Không trả lỗi chi tiết cho client, log lỗi ở server.
  - Cập nhật dependencies thường xuyên, kiểm tra lỗ hổng với `npm audit`.

## Authentication & Authorization Security
- **JWT Security:**
  ```js
  // Sử dụng secret mạnh, ngẫu nhiên
  const jwt = require('jsonwebtoken');
  const token = jwt.sign(payload, process.env.JWT_SECRET, { 
    expiresIn: '15m',
    issuer: 'yourapp',
    audience: 'web'
  });
  
  // Verify token
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  ```
- **Password Security:**
  ```js
  const bcrypt = require('bcrypt');
  
  // Hash password với salt rounds cao
  const hashPassword = async (password) => {
    return await bcrypt.hash(password, 12);
  };
  
  // Verify password
  const verifyPassword = async (password, hash) => {
    return await bcrypt.compare(password, hash);
  };
  ```
- **Session Security:**
  ```js
  app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: true, // HTTPS only
      httpOnly: true, // Không truy cập từ JS
      maxAge: 1800000, // 30 phút
      sameSite: 'strict'
    }
  }));
  ```

## CORS Security
```js
const cors = require('cors');
app.use(cors({
  origin: ['https://yourdomain.com'], // Chỉ định domain cụ thể
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

## Rate Limiting & DoS Protection
```js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false
});

app.use('/api/', limiter);
```

## Encryption & Data Protection
- **Environment Variables:**
  ```js
  // Không hardcode secrets
  const dbUrl = process.env.DATABASE_URL;
  const apiKey = process.env.API_KEY;
  ```
- **Data Encryption:**
  ```js
  const crypto = require('crypto');
  
  const encrypt = (text) => {
    const algorithm = 'aes-256-gcm';
    const key = crypto.scryptSync(process.env.ENCRYPTION_KEY, 'salt', 32);
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(algorithm, key, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return { encrypted, iv: iv.toString('hex') };
  };
  ```

## Security Headers
```js
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

## File Upload Security
```js
const multer = require('multer');
const path = require('path');

const upload = multer({
  dest: 'uploads/',
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  },
  fileFilter: (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|pdf/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (mimetype && extname) {
      return cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  }
});
```

## Security Best Practices
1. **Principle of Least Privilege:** Chỉ cấp quyền tối thiểu cần thiết
2. **Defense in Depth:** Nhiều lớp bảo vệ
3. **Fail Securely:** Khi lỗi xảy ra, mặc định từ chối truy cập
4. **Keep Software Updated:** Luôn cập nhật phiên bản mới nhất
5. **Log và Monitor:** Ghi log hoạt động, theo dõi bất thường
6. **Input Validation:** Luôn validate dữ liệu đầu vào
7. **Use HTTPS:** Mã hóa dữ liệu truyền tải
8. **Regular Security Audits:** Kiểm tra bảo mật định kỳ
