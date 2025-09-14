# 4. Built-in Modules
- **fs (File System)**: Đọc/ghi file, thao tác với hệ thống file.
  - Ví dụ:
    ```js
    const fs = require('fs');
    fs.readFile('file.txt', 'utf8', (err, data) => {
      if (err) throw err;
      console.log(data);
    });
    ```

- **http**: Tạo server HTTP, xây dựng web server cơ bản.
  - Ví dụ:
    ```js
    const http = require('http');
    const server = http.createServer((req, res) => {
      res.end('Hello Node.js');
    });
    server.listen(3000);
    ```

- **path**: Xử lý đường dẫn file, join, resolve, basename, extname...
  - Ví dụ:
    ```js
    const path = require('path');
    const filePath = path.join(__dirname, 'file.txt');
    console.log(filePath);
    ```

- **os**: Lấy thông tin hệ điều hành (CPU, memory, platform, ...).
  - Ví dụ:
    ```js
    const os = require('os');
    console.log(os.platform(), os.cpus().length);
    ```

- **events**: Tạo và quản lý các sự kiện (EventEmitter pattern).
  - Ví dụ:
    ```js
    const EventEmitter = require('events');
    const emitter = new EventEmitter();
    emitter.on('greet', name => console.log('Hello', name));
    emitter.emit('greet', 'Node.js');
    ```

- **stream**: Xử lý dữ liệu lớn theo luồng (streaming), đọc/ghi file lớn, network...
  - Ví dụ:
    ```js
    const fs = require('fs');
    const readStream = fs.createReadStream('bigfile.txt');
    readStream.on('data', chunk => {
      console.log('Received:', chunk.length, 'bytes');
    });
    ```

- **url**: Phân tích và xử lý URL
  - Ví dụ:
    ```js
    const url = require('url');
    const myURL = url.parse('https://example.com/path?query=123');
    console.log(myURL.hostname); // example.com
    console.log(myURL.query); // query=123
    
    // ES6 URL API
    const myURL2 = new URL('https://example.com/path?query=123');
    console.log(myURL2.hostname); // example.com
    ```

- **querystring**: Xử lý query string trong URL
  - Ví dụ:
    ```js
    const querystring = require('querystring');
    const params = querystring.parse('name=John&age=30');
    console.log(params); // { name: 'John', age: '30' }
    
    const str = querystring.stringify({ name: 'Jane', age: 25 });
    console.log(str); // name=Jane&age=25
    ```

- **crypto**: Mã hóa, hash, tạo token bảo mật
  - Ví dụ:
    ```js
    const crypto = require('crypto');
    
    // Hash MD5/SHA256
    const hash = crypto.createHash('sha256')
                      .update('password123')
                      .digest('hex');
    console.log(hash);
    
    // Tạo random string
    const randomBytes = crypto.randomBytes(16).toString('hex');
    console.log(randomBytes);
    ```

- **util**: Các tiện ích hỗ trợ (promisify, inspect, format...)
  - Ví dụ:
    ```js
    const util = require('util');
    const fs = require('fs');
    
    // Promisify callback function
    const readFileAsync = util.promisify(fs.readFile);
    readFileAsync('file.txt', 'utf8')
      .then(data => console.log(data))
      .catch(err => console.error(err));
    
    // Format string
    const formatted = util.format('Hello %s, you are %d years old', 'John', 25);
    console.log(formatted);
    ```

- **process**: Thông tin về process hiện tại, environment variables
  - Ví dụ:
    ```js
    // Environment variables
    console.log(process.env.NODE_ENV);
    
    // Command line arguments
    console.log(process.argv);
    
    // Exit process
    process.on('exit', code => {
      console.log('Process exiting with code:', code);
    });
    
    // Memory usage
    console.log(process.memoryUsage());
    ```

- **buffer**: Xử lý dữ liệu binary
  - Ví dụ:
    ```js
    // Tạo buffer từ string
    const buf1 = Buffer.from('Hello World', 'utf8');
    console.log(buf1); // <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>
    
    // Tạo buffer với size cố định
    const buf2 = Buffer.alloc(10);
    buf2.write('Node.js');
    console.log(buf2.toString()); // Node.js
    
    // Buffer concat
    const buf3 = Buffer.concat([buf1, buf2]);
    ```

- **child_process**: Chạy commands và processes con
  - Ví dụ:
    ```js
    const { exec, spawn, fork } = require('child_process');
    
    // Execute command
    exec('ls -la', (error, stdout, stderr) => {
      if (error) return console.error(error);
      console.log(stdout);
    });
    
    // Spawn new process
    const ls = spawn('ls', ['-la']);
    ls.stdout.on('data', data => console.log(data.toString()));
    
    // Fork Node.js process
    const child = fork('./worker.js');
    child.send({ message: 'Hello child' });
    ```

- **cluster**: Tạo multi-process để tận dụng multi-core CPU
  - Ví dụ:
    ```js
    const cluster = require('cluster');
    const numCPUs = require('os').cpus().length;
    
    if (cluster.isMaster) {
      // Fork workers
      for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
      }
      
      cluster.on('exit', (worker) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork();
      });
    } else {
      // Worker process
      const http = require('http');
      http.createServer((req, res) => {
        res.end('Hello from worker ' + process.pid);
      }).listen(3000);
    }
    ```

## Câu hỏi phỏng vấn thường gặp:

### 1. **fs module**
- Q: Sự khác biệt giữa `fs.readFile()` và `fs.readFileSync()`?
- A: `readFile()` là async (non-blocking), `readFileSync()` là sync (blocking). Nên dùng async trong production.

### 2. **events module**
- Q: EventEmitter hoạt động như thế nào? Có bao nhiều listeners tối đa?
- A: EventEmitter cho phép emit và listen events. Default max listeners = 10, có thể config bằng `setMaxListeners()`.

### 3. **stream module**
- Q: 4 loại stream trong Node.js là gì?
- A: Readable, Writable, Duplex, Transform. Dùng để xử lý dữ liệu lớn mà không cần load hết vào memory.

### 4. **buffer**
- Q: Buffer khác String như thế nào?
- A: Buffer lưu trữ raw binary data, String lưu trữ text với encoding. Buffer có fixed size, String có thể thay đổi.

### 5. **crypto**
- Q: Làm sao hash password an toàn?
- A: Dùng `bcrypt` hoặc `scrypt`, không dùng MD5/SHA1. Thêm salt để chống rainbow table attacks.

### 6. **child_process**
- Q: Khi nào dùng `exec()` vs `spawn()` vs `fork()`?
- A: 
  - `exec()`: Chạy shell commands ngắn
  - `spawn()`: Chạy commands lâu, stream I/O
  - `fork()`: Chạy Node.js scripts khác

### 7. **cluster**
- Q: Cluster module giải quyết vấn đề gì?
- A: Node.js chỉ chạy single-thread, cluster giúp tạo nhiều processes để tận dụng multi-core CPU, tăng performance.
