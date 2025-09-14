# 6. Xử lý lỗi
- **try/catch với async/await**: Dùng để bắt lỗi khi sử dụng async/await với Promise.
  - Ví dụ:
    ```js
    async function readFileAsync() {
      try {
        const data = await readFilePromise('file.txt');
        console.log(data);
      } catch (err) {
        console.error('Lỗi:', err);
      }
    }
    ```

- **Xử lý lỗi callback (err-first callback)**: Theo convention của Node.js, callback luôn nhận tham số đầu tiên là error (nếu có lỗi), tiếp theo là kết quả.
  - Ví dụ:
    ```js
    fs.readFile('file.txt', 'utf8', (err, data) => {
      if (err) {
        console.error('Lỗi:', err);
        return;
      }
      console.log(data);
    });
    ```

- **Global error handler**: Bắt các lỗi không được xử lý ở cấp toàn cục để tránh app bị crash đột ngột.
  - Ví dụ:
    ```js
    process.on('uncaughtException', (err) => {
      console.error('Uncaught Exception:', err);
      // Có thể log lỗi, gửi alert, hoặc shutdown app an toàn
    });
    process.on('unhandledRejection', (reason, promise) => {
      console.error('Unhandled Rejection:', reason);
    });
    ```

- **Custom Error Classes**: Tạo các lỗi custom để phân loại và xử lý theo từng loại.
  - Ví dụ:
    ```js
    class ValidationError extends Error {
      constructor(message, field) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
        this.statusCode = 400;
      }
    }

    class DatabaseError extends Error {
      constructor(message, query) {
        super(message);
        this.name = 'DatabaseError';
        this.query = query;
        this.statusCode = 500;
      }
    }

    // Sử dụng
    function validateUser(user) {
      if (!user.email) {
        throw new ValidationError('Email is required', 'email');
      }
    }
    ```

- **Error Handling trong Express.js**:
  - Middleware xử lý lỗi:
    ```js
    // Error handling middleware (phải có 4 parameters)
    app.use((err, req, res, next) => {
      console.error(err.stack);
      
      if (err instanceof ValidationError) {
        return res.status(400).json({
          error: err.message,
          field: err.field
        });
      }
      
      if (err instanceof DatabaseError) {
        return res.status(500).json({
          error: 'Database error occurred'
        });
      }
      
      // Default error
      res.status(500).json({ error: 'Internal server error' });
    });

    // Async route wrapper để bắt lỗi
    const asyncWrapper = (fn) => {
      return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
      };
    };

    app.get('/users/:id', asyncWrapper(async (req, res) => {
      const user = await User.findById(req.params.id);
      if (!user) {
        throw new ValidationError('User not found', 'id');
      }
      res.json(user);
    }));
    ```

- **Promise Error Handling**:
  - Các cách xử lý lỗi với Promise:
    ```js
    // Method 1: .catch()
    fetchUserData(id)
      .then(user => console.log(user))
      .catch(err => console.error('Error:', err));

    // Method 2: async/await
    try {
      const user = await fetchUserData(id);
      console.log(user);
    } catch (err) {
      console.error('Error:', err);
    }

    // Method 3: Promise.allSettled() - không throw error
    const results = await Promise.allSettled([
      fetchUser(1),
      fetchUser(2),
      fetchUser(3)
    ]);

    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`User ${index}:`, result.value);
      } else {
        console.error(`Error ${index}:`, result.reason);
      }
    });
    ```

- **Error Logging và Monitoring**:
  ```js
  const winston = require('winston');

  const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.errors({ stack: true }),
      winston.format.json()
    ),
    transports: [
      new winston.transports.File({ filename: 'error.log', level: 'error' }),
      new winston.transports.File({ filename: 'combined.log' })
    ]
  });

  // Error handling với logging
  process.on('uncaughtException', (err) => {
    logger.error('Uncaught Exception:', err);
    process.exit(1);
  });

  process.on('unhandledRejection', (reason, promise) => {
    logger.error('Unhandled Rejection at:', promise, 'reason:', reason);
  });
  ```

- **Graceful Shutdown**:
  ```js
  const server = app.listen(3000);

  const gracefulShutdown = (signal) => {
    console.log(`Received ${signal}, shutting down gracefully`);
    
    server.close(() => {
      console.log('HTTP server closed');
      
      // Close database connections
      mongoose.connection.close(() => {
        console.log('Database connection closed');
        process.exit(0);
      });
    });

    // Force shutdown after timeout
    setTimeout(() => {
      console.error('Could not close connections in time, forcefully shutting down');
      process.exit(1);
    }, 10000);
  };

  process.on('SIGTERM', gracefulShutdown);
  process.on('SIGINT', gracefulShutdown);
  ```

- **Circuit Breaker Pattern**:
  ```js
  class CircuitBreaker {
    constructor(request, options = {}) {
      this.request = request;
      this.state = 'CLOSED';
      this.failureCount = 0;
      this.nextAttempt = Date.now();
      this.threshold = options.threshold || 5;
      this.timeout = options.timeout || 10000;
    }

    async call(...args) {
      if (this.state === 'OPEN') {
        if (Date.now() < this.nextAttempt) {
          throw new Error('Circuit breaker is OPEN');
        }
        this.state = 'HALF-OPEN';
      }

      try {
        const result = await this.request(...args);
        this.onSuccess();
        return result;
      } catch (err) {
        this.onFailure();
        throw err;
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

  // Sử dụng
  const apiCall = new CircuitBreaker(async (url) => {
    const response = await fetch(url);
    if (!response.ok) throw new Error('API call failed');
    return response.json();
  });
  ```

- **Error Handling Best Practices**:
  ```js
  // 1. Fail Fast - Validate early
  function processUser(user) {
    if (!user || !user.id) {
      throw new ValidationError('User ID is required');
    }
    // Process user...
  }

  // 2. Return errors, don't throw (functional approach)
  async function fetchUserSafely(id) {
    try {
      const user = await User.findById(id);
      return { success: true, data: user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // 3. Error boundaries trong production
  const safeExecute = async (fn) => {
    try {
      return await fn();
    } catch (error) {
      logger.error('Safe execution failed:', error);
      return null; // hoặc default value
    }
  };
  ```

## Câu hỏi phỏng vấn thường gặp:

### 1. **uncaughtException vs unhandledRejection**
- Q: Sự khác biệt và cách xử lý?
- A: uncaughtException cho sync errors, unhandledRejection cho Promise rejections. Nên log và graceful shutdown.

### 2. **Express Error Handling**
- Q: Error middleware trong Express hoạt động như thế nào?
- A: Middleware có 4 parameters (err, req, res, next), phải đặt cuối cùng, dùng next(err) để trigger.

### 3. **Promise Error Handling**
- Q: Điều gì xảy ra khi Promise reject không được catch?
- A: Tạo unhandledRejection event, có thể crash app nếu không handle.

### 4. **Error Classification**
- Q: Phân loại errors trong Node.js?
- A: 
  - Operational errors: Network, file system, validation
  - Programmer errors: Logic bugs, wrong types
  - System errors: Out of memory, disk full

### 5. **Error Recovery Strategies**
- Q: Các chiến lược recover từ errors?
- A: Retry with exponential backoff, circuit breaker, fallback responses, graceful degradation.

### 6. **Error Logging**
- Q: Best practices cho error logging?
- A: Structured logging, different log levels, correlation IDs, avoid logging sensitive data.

### 7. **Testing Error Handling**
- Q: Làm sao test error scenarios?
- A: Mock dependencies để throw errors, test cả positive và negative cases, integration tests.

### 8. **Production Error Handling**
- Q: Monitoring và alerting trong production?
- A: APM tools (New Relic, DataDog), health checks, metrics, automated alerts, error tracking.
